<!-- TOC -->

- [COVID-19 Sicilia](#covid-19-sicilia)
  - [Perché questo spazio](#perch%c3%a9-questo-spazio)
  - [File di progetto QGIS](#file-di-progetto-qgis)
  - [Cosa c'è in questo repo](#cosa-c%c3%a8-in-questo-repo)
  - [Espressione usata](#espressione-usata)
  - [Virtual File Format di GDAL/OGR](#virtual-file-format-di-gdalogr)
    - [regioni](#regioni)
    - [province](#province)
  - [Compositore di stampe](#compositore-di-stampe)
    - [Atlas](#atlas)
  - [Caratteristiche utilizzate nel progetto](#caratteristiche-utilizzate-nel-progetto)
  - [Riferimenti utili](#riferimenti-utili)

<!-- /TOC -->

# COVID-19 Sicilia
Raccolta dati sul **COVID-19** per scopo informativo.

## Perché questo spazio

Progetto QGIS per la visualizzazione dei dati COVID-19 attraverso un atlas con grafici dinamici - regioni e province ISTAT - fonte : https://github.com/pcm-dpc/COVID-19

## File di progetto QGIS

Il file di progetto QGIS (COVID19_3857_provaut) utilizza come fonte dati il file dpc-covid19-ita-regioni.csv presente nel repository ufficiale del PCM-DPC tramite Protocollo HTTPS, quindi il file si aggiorna automaticamente:

![](imgs/https.png)

---
NB: il file di progetto è stato realizzato con [QGIS 3.12 București](https://qgis.org/it/site/) e Plugin [DataPlotly 3.5](https://github.com/ghtmtt/DataPlotly)

---

## Cosa c'è in questo repo

- cartella `imgs` contiene le immagini utilizzate nel progetto .qgs;
- cartella `risorse` contiene i file utilizzati nel progetto, come:
  - shapefile `reg_provaut3857.*` limiti amministrativi regionali ISTAT 2019, EPSG:3857;
  - shapefile* `reg_provaut3857.*` limiti amministrativi regionali ISTAT 2019 con Prov. Autonome Trento e Bolzano, EPSG:3857;
  - file `codid19-regioni.vrt` Virtual File Format GDAL/OGR con file CSV raw da GitHub, con geometry Point;
  - file `dpc-covid19-ita-province.vrt` Virtual File Format GDAL/OGR con file CSV raw da GitHub, no geometry;
  - file `dpc-covid19-ita-regioni.vrt` Virtual File Format GDAL/OGR con file CSV raw da GitHub, no geometry;;
  - file `regione.xml` di configurazione grafici atlas;
  - file `province.xml` di configurazione grafici atlas;
  - file `world_map.gpkg` geopackage con la world map;
  - file `codid19-andamento_nazione.vrt` Virtual File Format GDAL/OGR con file CSV raw da GitHub;
  - file `CHANGELOG.md` file con le novità;
- cartella `PDF` stampe giornaliere dell'Atlas;
- file `progetto_sicilia_3857.qgs` è il file di progetto QGIS in formato `.qgs`, EPSG:3857;
- file `license` è il file che definisce la licenza del repository;
- file `README.md` è questo file, con le info.

[↑ torna su ↑](#perch%c3%a9-questo-spazio)

## Espressione usata

Per etichettare le province e la regione, con valore attuale e incremento giornaliero:

```
 relation_aggregate( 
 relation:='rel_reg',
 aggregate:='array_agg',
 expression:="totale_casi")  [-1] 
 || ' (' ||
if ( ( to_int( relation_aggregate( 
 relation:='rel_reg',
 aggregate:='array_agg',
 expression:="totale_casi")  [-1] ) 
  - 
 to_int( relation_aggregate( 
 relation:='rel_reg',
 aggregate:='array_agg',
 expression:="totale_casi") [-2 ] ) ) > 0, '+', '' ) 
 ||  
  ( to_int( relation_aggregate( 
 relation:='rel_reg',
 aggregate:='array_agg',
 expression:="totale_casi")  [-1] ) 
  - 
 to_int( relation_aggregate( 
 relation:='rel_reg',
 aggregate:='array_agg',
 expression:="totale_casi") [-2 ] ) ) 
 || ') '
```


![](./imgs/etichetta_incremento.png)

[↑ torna su ↑](#perch%c3%a9-questo-spazio)

## Virtual File Format di GDAL/OGR

### regioni

link utile: <https://gdal.org/drivers/vector/vrt.html#virtual-file-format>

```xml
<OGRVRTDataSource>
<OGRVRTLayer name="dpc-covid19-ita-regioni">
    <SrcDataSource relativeToVRT="0">/vsicurl/https://raw.githubusercontent.com/pcm-dpc/COVID-19/master/dati-regioni/dpc-covid19-ita-regioni.csv</SrcDataSource>
    <Field name="data" type="String" />
    <Field name="lat" type="Real" />
    <Field name="long" type="Real" />
    <Field name="stato" type="String" />
    <Field name="codice_regione" type="String" />
    <Field name="denominazione_regione" type="String" />
	  <Field name="ricoverati_con_sintomi" type="Integer" />
    <Field name="terapia_intensiva" type="Integer" />
    <Field name="totale_ospedalizzati" type="Integer" />
    <Field name="isolamento_domiciliare" type="Integer" />
    <Field name="totale_attualmente_positivi" type="Integer" />
    <Field name="nuovi_attualmente_positivi" type="Integer" />
    <Field name="dimessi_guariti" type="Integer" />
    <Field name="deceduti" type="Integer" />
    <Field name="totale_casi" type="Integer" />
    <Field name="tamponi" type="Integer" />
</OGRVRTLayer>
</OGRVRTDataSource>
```

### province

```xml
<OGRVRTDataSource>
<OGRVRTLayer name="dpc-covid19-ita-province">
    <SrcDataSource relativeToVRT="0">/vsicurl/https://raw.githubusercontent.com/pcm-dpc/COVID-19/master/dati-province/dpc-covid19-ita-province.csv</SrcDataSource>
    <Field name="data" type="String" />
	<Field name="stato" type="String" />
	<Field name="codice_regione" type="String" />
	<Field name="denominazione_regione" type="String" />
	<Field name="codice_provincia" type="String" />
	<Field name="denominazione_provincia" type="String" />
	<Field name="sigla_provincia" type="String" />
	<Field name="totale_casi" type="Integer" />
</OGRVRTLayer>
</OGRVRTDataSource>
```


- per un quadro sinottico del file

```
ogrinfo codid19-regioni_dw.vrt dpc-covid19-ita-regioni -summary
```

- per leggere il file con OGR:

```
ogrinfo codid19-regioni_dw.vrt dpc-covid19-ita-regioni
```

per ottenere il nome layer corretto

```
ogrinfo -ro -al -q CSV:/vsicurl/https://raw.githubusercontent.com/pcm-dpc/COVID-19/master/dati-regioni/dpc-covid19-ita-regioni.csv
```

- per usarlo in QGIS:

![](imgs/https_vrt.png)

## Compositore di stampe

### Atlas

Vettore di copertura : layer `reg_provaut3857`, Font [`TRUENO`](https://www.wfonts.com/font/trueno)

![](imgs/atlas_vl_01.png)

**Gif animata:**

![](./stampe/covid17_atlas_prov.gif)

[↑ torna su ↑](#perch%c3%a9-questo-spazio)


## Caratteristiche utilizzate nel progetto

1. Shapefile, Geopackage, CSV, CSV remoti;
2. Join, Relazioni;
3. Atlas con grafici dinamici (Plugin DataPlotly);
4. Layout di stampa Andamento nazionale (Plugin DataPlotly);
5. Visualizzazione immagini remote (Stemmi);
6. Tematizzazione tramite regole;
7. Calcolo valori incrementali giornalieri tramite espressioni;
8. Temi mappe per Atlas;
9. Tabella in relazione nell'Atlas e formattazione condizionale;
10. Panoramica con Generatore di geometria;
11. Etichette con valori raggruppati e incrementali.
12. Decorazioni: Copyright, Immagine e estensioni layout

[↑ torna su ↑](#perch%c3%a9-questo-spazio)

## Riferimenti utili

- **QGIS** : <https://qgis.org/it/site/>
- **Plugin DataPlotly** : <https://github.com/ghtmtt/DataPlotly>
- **Fonti dati PCM-DPC** : <https://github.com/pcm-dpc/COVID-19>
- **CONFINI DELLE UNITÀ AMMINISTRATIVE A FINI STATISTICI AL 1 GENNAIO 2019** : <https://www.istat.it/it/archivio/222527>
- **Word Map** : <https://www.naturalearthdata.com/downloads/10m-cultural-vectors/>
- **Stemmi Regioni Italiane** : <https://it.wikipedia.org/wiki/Stemmi_delle_regioni_italiane>;
- **Font Trueno** : <https://www.wfonts.com/font/trueno>
- **Visual Style Guide** : <https://www.qgis.org/en/site/getinvolved/styleguide.html#trueno-fonts>
- **Visual Studio Code** : <https://code.visualstudio.com/>

![](./imgs/istat88x31.png)
**NB:** Tutti i dati prodotti dall’Istituto nazionale di statistica (ISTAT) sono rilasciati sotto [licenza Creative Commons (CC BY 3.0 IT)](https://www.istat.it/it/note-legali): è possibile riprodurre, distribuire, trasmettere e adattare liberamente dati e analisi dell’Istituto nazionale di statistica, anche a scopi commerciali, a **condizione che venga citata la fonte**.

[↑ torna su ↑](#perch%c3%a9-questo-spazio)

