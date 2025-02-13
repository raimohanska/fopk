# Funktionaalisempi ohjelmointi proseduraalisella kielellä

Funktionaalisessa ohjelmoinnissa rakennetaan ohjelmisto siten, että se koostuu tilattomista funktioista, joiden tulos riippuu pelkästään syötteestä. Sen pohjana on lambda-kalkyyli ja matematiikasta tuttu funktion käsite. Erona proseduraaliseen ohjelmointiin on se, että funktionaalisessa ohjelmoinnissa arvioidaan lausekkeita käskyjen antamisen sijaan. 

## Funktionaalisen ohjelmoinnin edut

Proseduulisessa ohjelmoinnissa mikä tahansa käsky voi muokata ohjelman tilaa. Tätä kutsutaan sivuvaikutukseksi (side-effect). Ohjelmiston ymmärtämisen kannalta rajoittamattomat sivuvaikutukset ovat haitallisia. Esimerkiksi säikeisessä (multi-threaded) ohjelmistossa käsiteltävä tieto voi muuttua kesken proseduurien suoritusten, mikä voi johtaa vaikeasti selvitettäviin ongelmiin. Koska funktionaalisessa ohjelmoinnissa funktion paluuarvo riippuu vain syötteestä ja tieto on oletusarvoisesti muuttumatonta, tämän tyyppisiä ongelmia ei esiinny.

Soveltamalla muutamia funktionaalisen ohjelmoinnin käsitteitä voidaan proseduraalisten tai olio-pohjaisten ohjelmistojen rakennetta selkiyttää ja vähentää virheitä merkittävästi. Lisäksi ohjelmiston rinnakkaistaminen helpottuu, koska tietoa ei tarvitse lukita säikeiden välillä.

## Funktionaaliset ohjelmointitekniikat

Muutamia funktionaalisia ohjelmointitekniikoita voidaan hyödyntää myös proseduraalisissa ohjelmointikielissä. Seuraavissa kappaleissa esittelemme muutamia niistä ja esimerkkejä sekä Javalla että C++:lla.

Koska näitä funktionaalisten ohjelmointikielien ominaisuuksia ei ole suoraan rakennettu näihin kieliin, monet tekniikoista saattavat vaikuttaa oudoilta tai jopa tarkoituksettomilta, mutta niiden hyödyntäminen johtaa moniin samoihin etuihin joista funktionaalisen ohjelmointikielten ohjelmoijat nauttivat.

### Tilattomat funktiot (Stateless function)

Proseduraalisessa ohjelmointikielissä metodit voidaan kirjoittaa siten, että ne eivät muokkaa omaa syötettään tai ohjelman tilaa.

*Esimerkki Javalla*

```java
package functional.java;

import java.util.ArrayList;
import java.util.List;

public class Filter {
    public List<String> apply(List<String> values, Predicate<String> predicate) {
        ArrayList<String> output = new ArrayList<String>();
        for (String string : values) {
            if (predicate.apply(string)) {
                output.add(string);
            }
        }
        return output;
    }

    public interface Predicate<T> {
        public boolean apply(T input);
    }
}
```

*Esimerkki C++:lla*

```cpp
#include <boost/function.hpp>
#include <boost/bind.hpp>
#include <boost/foreach.hpp>

using namespace std;
using namespace boost;

bool exact(string& value, string expected)
{
    return value == expected;
}

const vector<string> filter(const vector<string>& values,
                            function<bool(string)> predicate)
{
    vector<string> output;
    BOOST_FOREACH (string value, values)
    {
        if (predicate(value))
            output.push_back(value);
    }
    return output;
}
```

Edelläolevassa esimerkissä syötettä ei suoraan muokata, vaan metodissa palautetaan uusi lista predikaatin täyttämällä ehdolla.

### Muuttumaton data (Immutable data)

Yksi helpoimpia tapoja vähentää sivuvaikutuksien muodostumista on estää datan suora muokkaaminen. Olio-kielissä tämä edellyttää sellaista olioiden tekemistä, jotka eivät anna muokata omia muuttujiaan.

#### Javalla

Tyypillinen Java-bean-rakenne ohjaa väärään suuntaan ja sen sijaan kannattaa suosia final-avainsanaa. Muuttumattomat oliot vaativat avukseen apuluokkia, jotta niiden muodostaminen onnistuu kivuttomasti. Usein käytetty tapa on  rakentaja-olio (Builder-pattern).

*muuttumaton dataluokka*

```java
package functional.java;

public class Address implements ContactInformation {
	private final String streetAddress;
	private final String postCode;
	private final String postOffice;

	public Address(String streetAddress, String postCode, String postOffice) {
		this.streetAddress = streetAddress;
		this.postCode = postCode;
		this.postOffice = postOffice;
	}

	@Override
	public String getStreetAddress() {
		return streetAddress;
	}

	@Override
	public String getPostCode() {
		return postCode;
	}

	@Override
	public String getPostOffice() {
		return postOffice;
	}
}
```

*rakentaja*

```java
package functional.java;

public class AddressBuilder {
	private String buildStreetAddress;
	private String buildPostCode;
	private String buildPostOffice;

	public AddressBuilder withAddress(String streetAddress) {
		buildStreetAddress = streetAddress;
		return this;
	}

	public AddressBuilder withPostCode(String postCode) {
		buildPostCode = postCode;
		return this;
	}

	public AddressBuilder withPostOffice(String postOffice) {
		buildPostOffice = postOffice;
		return this;
	}

	public Address build() {
		return new Address(buildStreetAddress, buildPostCode, buildPostOffice);
	}
}
```

Toinen tapa rakentaa muuttumattomia olioita on edustaja (Proxy). Sen sijaan että oliolla on omia muuttujia, se toimii näkymänä toisten olioiden tietosisältöön.

*edustajan rajapinta*

```java
package functional.java;

import java.io.Serializable;

public interface ContactInformation extends Serializable {
	public static final ContactInformation NO_CONTACT_INFORMATION = new NoContactInformation();

	String getStreetAddress();

	String getPostCode();

	String getPostOffice();

	public static final class NoContactInformation implements ContactInformation {
		@Override
		public String getStreetAddress() {
			return "";
		}

		@Override
		public String getPostCode() {
			return "";
		}

		@Override
		public String getPostOffice() {
			return "";
		}
	}
}
```

*edustaja*

```java
package functional.java;

import java.sql.ResultSet;
import java.sql.SQLException;

import javax.inject.Inject;

import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.ResultSetHandler;

public class DbAddress implements ContactInformation {
	@Inject
	private QueryRunner database;
	private final Integer id;

	public DbAddress(Integer id) {
		this.id = id;
	}

	@Override
	public String getStreetAddress() {
		return fetchDbContactInfo("streetAddress");
	}

	@Override
	public String getPostCode() {
		return fetchDbContactInfo("postCode");
	}

	@Override
	public String getPostOffice() {
		return fetchDbContactInfo("postOffice");
	}

	private String fetchDbContactInfo(String fieldName) {
		try {
			return database.query("select " + fieldName + " from contactinfo where id = ?", new FirstStringHandler(), id);
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}

	private static final class FirstStringHandler implements ResultSetHandler<String> {
		@Override
		public String handle(ResultSet rs) throws SQLException {
			if (rs.next()) {
				return rs.getString(0);
			}
			throw new SQLException("No result");
		}
	}
}
```

Huomaa, että ContactInformation-rajapinnalla on oma tyhjä vakio NO_CONTACT_INFORMATION, jota on hyvä käyttää sen sijaan että palauttaisi null-arvon. Tällöin null-tarkistuksien sijaan voidaan verrata suoraan NO_CONTACT_INFORMATION-vakioon.

Javassa ei ole sisäänrakennettua tapaa saada muuttumattomia tietorakenteita, kuten listoja (List) tai taulukkoja (Map). Tähän tarkoitukseen kannattaa käyttää esimerkiksi [Googlen guava-kirjastoa](http://code.google.com/p/guava-libraries/).

#### C++:lla

C++:ssa ei ole muuttumattomia tietorakenteita, mutta const-avainsanan käytöllä voidaan estää esimerkiksi syötteenä välitettävän listan muokkaaminen.

*Esimerkki C++:lla*

### Koostaminen (Composition)

Koostamisessa funktion palautusarvot sopivat suoraan seuraavan funktion syötteeksi. Tällä tavalla funktiota voidaan helposti ketjuttaa toisiinsa, sekä niistä tulee lyhyitä ja helposti uudelleenkäytettäviä.

Javassa ja C++:ssa koostaminen tehdään funktio-olioilla, jotka alustetaan syötteellä ja tuottavat saman tuloksen. Funktio-olioita voidaan antaa syötteeksi toisille funktio-olioille jolloin saadaan aikaan ns. korkean asteen funktioita.

#### Koostaminen Javalla

Funktion rajapinta on  yksinkertainen ja se löytyy mm. [guava-kirjastosta](http://code.google.com/p/guava-libraries/).

```java
package functional.java;

public interface Function<F, T> {
	public T apply(F input);
}
````

Voimme helposti saada aikaan vaikkapa välimuistin käyttämällä funktiota, joka ottaa funktioita syötteekseen.

```java
package functional.java;

import java.util.Map;

import com.google.common.base.Function;
import com.google.common.collect.MapMaker;

public class CachingFunction<F, T> implements Function<F, T> {
	private final Map<F, T> cache;

	public CachingFunction(final Function<F, T> source) {
		this.cache = new MapMaker().softValues().makeComputingMap(source);
	}

	@Override
	public T apply(final F input) {
		return cache.get(input);
	}

	public static <F, T> Function<F, T> cache(final Function<F, T> source) {
		return new CachingFunction<F, T>(source);
	}
}
```
Muutetaanpa edellisen kappaleen esimerkin tietokantahaku funktioksi.

*Rajapinta*

```java
package functional.java;

public interface ContactInfoFetcher {
	public String fetch(DbKey input);
}
```

*Toteutus*

```java
package functional.java;

import java.sql.ResultSet;
import java.sql.SQLException;

import javax.inject.Inject;

import org.apache.commons.dbutils.QueryRunner;
import org.apache.commons.dbutils.ResultSetHandler;

import com.google.common.base.Function;

public class DbContactInfoFetcher implements ContactInfoFetcher, Function<DbKey, String> {
	@Inject
	private QueryRunner database;

	@Override
	public String fetch(DbKey input) {
		return apply(input);
	}

	private static final class FirstStringHandler implements ResultSetHandler<String> {
		@Override
		public String handle(ResultSet rs) throws SQLException {
			if (rs.next()) {
				return rs.getString(0);
			}
			throw new SQLException("No result");
		}
	}

	@Override
	public String apply(DbKey input) {
		try {
			return database.query("select " + input.getFieldName() + " from contactinfo where id = ?", new FirstStringHandler(),
				input.getId());
		} catch (SQLException e) {
			throw new RuntimeException(e);
		}
	}
}
```

*Toteutus välimuistilla*

```java
package functional.java;

import com.google.common.base.Function;

public class CachedContactInfoFetcher implements ContactInfoFetcher {
	private final Function<DbKey, String> cache = CachingFunction.cache(new DbContactInfoFetcher());

	@Override
	public String fetch(DbKey input) {
		return cache.apply(input);
	}
}
```

### Muunnokset (Transformation)

Muunnoksessa data muutetaan seuraavan funktion tarvitsemaan muotoon muuttamatta alkuperäistä dataa.

#### Javalla

*muuntajaluokka*

```java
package functional.java;

import static functional.java.ContactInformation.NO_CONTACT_INFORMATION;

import java.util.Arrays;
import java.util.List;

public class AddressTransformer implements Function<String, ContactInformation> {
	@Override
	public ContactInformation apply(String input) {
		final List<String> addressLines = addressLines(input);
		try {
			return new AddressBuilder().withAddress(firstItem(addressLines))
				.withPostCode(firstItem(postCodeAndOffice(addressLines)))
				.withPostOffice(secondItem(postCodeAndOffice(addressLines)))
				.build();
		} catch (ArrayIndexOutOfBoundsException e) {
			return NO_CONTACT_INFORMATION;
		}
	}

	private List<String> addressLines(String input) {
		return Arrays.asList(input.split("\n"));
	}

	private List<String> postCodeAndOffice(final List<String> addressLines) {
		return Arrays.asList(secondItem(addressLines).split(" "));
	}

	private String secondItem(final List<String> postCodeAndOffice) {
		return postCodeAndOffice.get(1);
	}

	private String firstItem(final List<String> addressLines) {
		return addressLines.get(0);
	}
}
```

*muuntajaluokan testi*

```java
package functional.java;

import static functional.java.ContactInformation.NO_CONTACT_INFORMATION;
import static junit.framework.Assert.assertEquals;

import org.junit.Test;

public class AddressTransformerTest {
	private final static String ADDRESS = "Testitie 5\n00999 OLEMATON";
	private final static String MISSING_CITY = "Testitie 5\n00999OLEMATON";
	private final static String MISSING_SECOND_LINE = "FUBAR";

	@Test
	public void WithAddress() {
		final ContactInformation address = new AddressTransformer().apply(ADDRESS);
		assertEquals("Testitie 5", address.getStreetAddress());
		assertEquals("00999", address.getPostCode());
		assertEquals("OLEMATON", address.getPostOffice());
	}

	@Test
	public void WithMissingCity() {
		final ContactInformation address = new AddressTransformer().apply(MISSING_CITY);
		assertEquals(NO_CONTACT_INFORMATION, address);
	}

	@Test
	public void WithMissingSecondLine() {
		final ContactInformation address = new AddressTransformer().apply(MISSING_SECOND_LINE);
		assertEquals(NO_CONTACT_INFORMATION, address);
	}
}
```

## Yhteenveto

Funktionaalisten ohjelmointitekniikoiden käyttö vaatii ohjelmoijalta kurinalaisuutta ja pieteettiä. Palkintona saadaan kuitenkin helpommin testattava ja paremmin muutoksia sietävä ohjelmisto.
