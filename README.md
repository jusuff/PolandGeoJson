# [PolandGeoJson](https://github.com/jusuff/PolandGeoJson/)

Set of Polish administrative borders in geoJson format.

This set contains separate files for each of the 3 administrative levels (municipalities/gminy, counties/powiaty, voivodships/województwa) and a file for the country borders. The borders for each level are overlaping (the borders of counties, voivodships and the country were generated by dissolving borders of municipalities).

This data was prepared for the project <https://atlasnienawisci.pl>. Because I did not find such data when I was doing Atlas - I share it, maybe it will make life easier for someone.

## Sources

The borders were prepared on the basis of the following sources and tools:
* administrative boundaries of municipalities downloaded in the shapefile from <https://gis-support.pl/granice-administracyjne/>
* simplifying boundaries and converting from shatefile to geoJson using <https://mapshaper.org/>
* dissolving municipalities borders to obtain the borders of counties, voivodships and the country using the [dissolve library](https://github.com/deoxxa/dissolve)


## File syntax

The files contain the standard geoJson feature collections. For each element, the properties consist of two elements:
* name - unit name
* terc - unit TERC code

The primary identifier is TERC. The code base (TERC basic) and code syntax are available on the [GUS website](http://eteryt.stat.gov.pl/eTeryt/rerezent_teryt/udostepwanie_danych/baza_teryt/uzytkownicy_indywidualni/pobarcie/pliki_pelne.aspx?contrast=default)

## How to generate borders yourself?

The borders provided here are greatly simplified (although the file with the borders of municipalities is still over 3Mb). If you need more precise borders, you can generate them yourself. Below is a description of how I generated provided data:

1. Download administrative borders from <https://gis-support.pl/granice-administracyjne/>. There are borders available, in shapefile format, for all administrative levels - all you need is municipalities. On their basis, you can generate the boundaries of counties, voivodships and country, maintaining a perfect match.
2. Convert the shapefile to geoJson format and simplify the boundaries (if you need). I used <https://mapshaper.org/> - it allows both simplification and conversion to geoJson.
3. On the basis of municipalities boundaries generate boundaries of counties, voivodships and the country using the dissolve library. This way, even if you simplify the municipal borders, you will keep them perfectly aligned. Use the TERC database to connect municipalities into counties and counties into voivodships. At the end of this page you can find example of script to dissolve borders.
4. Done - You have the boundary sets of the desired accuracy that fit together perfectly.

## Podgląd

### Country

![Country borders](/data/poland.country.png) 

### Voivodships

![Voivodships borders](/data/poland.voivodeships.png) 

### Counties

![Counties borders](/data/poland.counties.png) 

### Municipalities

![Municipalities borders](/data/poland.municipalities.png) 

## Script to dissolve borders

Example of a script for generating counties boundaries based on municipalities boundaries. Data is connected using  TERC code (the code for a municipality begins with the code for the relevant county - see the GUS website for details).

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


