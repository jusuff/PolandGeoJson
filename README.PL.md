# PolandGeoJson

Zestaw granic administracyjnych Polski w formacie geoJson. 

Zestaw zawiera osobne pliki dla każdego z 3 poziomów administracyjnych (gminy/municipalities, powiaty/counties, województwa/voivodeships) oraz plik dla granic kraju. Granice dla każdego poziomu pokrywają się (granice powiatów, województ i kraju zostały wygenerowane przez łączenie granic gmin).

Dane zostały przygotowane dla projektu <https://atlasnienawisci.pl>. Ponieważ sam, robiąc Atlas, nie znalazłem takich danych - udostępniam, może komuś ułatwi to życie.

## Źrodła

Granice zostały wykonane w opraciu o następujące źrodła i narzędzia:
* granice administracyjne gmin pobrane w shapefile ze strony https://gis-support.pl/granice-administracyjne/
* uproszczenie granic i konwersja z shatefile do geoJson za pomocą https://mapshaper.org/
* łączenie granic gmin do uzyskania granic powiatów, województw i kraju za pomocą biblioteki [dissolve](https://github.com/deoxxa/dissolve)


## Składnia pliku

Pliki zawierają standardowe kolekcje feature-ów geoJson. Dla każdego elementu properties składają się z dwu elementów:
* name - nazwa jednostki
* terc - kod TERC jednostki

Podstawowym identyfikatorem jest TERC. Baza kodów (TERC podstawowy) i składnia kodu dostępne są na [stronie GUS](http://eteryt.stat.gov.pl/eTeryt/rejestr_teryt/udostepnianie_danych/baza_teryt/uzytkownicy_indywidualni/pobieranie/pliki_pelne.aspx?contrast=default)

## Jak samemu wygenerować granice?

Udostępnione tutaj granice są mocno uproszczone (choć nadal plik z granicami gmin ma ponad 3Mb). Jeśli potrzebujesz dokładniejszych granic, możesz wygenerować granice samodzielnie. Poniżej opis sposobu, w jakim wygenerowałem te dane, które udostępniam:

1. Pobierz granice administracyjne z https://gis-support.pl/granice-administracyjne/. Są tam dostępne, w formacie shapefile, granice dla wszystkich poziomów administracyjnych - potrzebujesz tylko gmin. Na ich bazie możesz wygenerować granice powiatów, województ i kraju, zachowując idealne dopasowanie.
2. Skonwertuj plik shapefile do formatu geoJson oraz ewentualnie uprość granice. Ja użyłem https://mapshaper.org/ - pozwala zarówno na uproszczenie jak i konwersję do geoJson. 
3. Na bazie granic gmin wygeneruj granice powiatów, województw i kraju używając bilbioleki dissolve. W ten sposób, nawet po uproszczeniu granic gmin, zachowasz idealne dopasowanie granic. Skorzystaj z bazy TERC do łączenia gmin w powiaty i powiatów w województwa. Na końcu przykład skryptu do łączenia granic.
4. Gotowe - masz zestawy granic, o porządanej dokłądności, idealnie do siebie pasujące.

## Podgląd

### Kraj

![Granice kraju](/data/poland.country.png) 

### Województwa

![Granice województw](/data/poland.voivodeships.png) 

### Powiaty

![Granice powiatów](/data/poland.counties.png) 

### Gminy

![Granice gmin](/data/poland.municipalities.png) 

## Skrypt do łączenia granic

Przykład skryptu do wygenerowania granic powiatów na podstawie granic gmin. Łączenie danych bazuje na kodzie TERC (kod dla gminy zaczyna się od kodu dla właściwego powiatu - patrz strona GUS).

```javascript
const fs = require('fs');
const dissolve = require('geojson-dissolve');

let dest = 'dissolved.counties.json',// save to
    geoData = JSON.parse(fs.readFileSync('municipalities.json')), // lower level borders
    tercData = JSON.parse(fs.readFileSync('counties.terc.json')); // list of counties in format [{terc: '0000'},{...}]

let dissvoledShapes = tercData.map(function(row) {
    let shapes = geoData.features.filter(function(item) {
            return item.properties.JPT_KOD_JE.startsWith(row.Terc);
        }),
        shape = dissolve(shapes),
        feature = {
            'type': 'Feature',
            'geometry': shape,
            'properties': {
                'terc': row.Terc,
                'name': row.nazwa,
            }
        };
    return feature;
});

let geoCollection = {'type':'FeatureCollection', 'features': dissvoledShapes};

fs.writeFile(dest, JSON.stringify(geoCollection), function(err) {
    if(err) {
        return console.log(err);
    }
});
```



