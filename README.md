# wind

Terminal wind scraper for [windguru.cz](https://www.windguru.cz) — live station
readings plus GFS forecast for Bat Galim (Haifa) and Caesarea.

```
$ wind

  Bat Galim    8.1 kt g11.9  ↑ S 176°  29°C  09:08
  Caesarea    11.8 kt g15.7  ↑ S 196°  27°C  09:09

  Bat Galim  today
      9  10  11  12  13  14  15  16  17  18  19  20  21  22  23
     12  15  17  17  16  16  16  15  13  12  14  13  12   8   8   <- avg kt
     15  16  18  19  18  18  17  16  15  14  15  14  13   9  10   <- gust
      ↑   ↗   ↗   ↗   ↗   ↗   ↗   ↗   ↗   ↗   ↗   →   ↘   →   →   <- blowing toward
   Sun 8-13g13 W  Mon 4-10g10 NW  Tue 3-11g11 N  Wed 5-12g12 N
```

Knots. Colour by strength: dim <8, green 8–14, yellow 15–22, red 23–31, magenta 32+.
Day summaries cover 08:00–19:00 local and show the range, the peak gust, and the
direction at that peak.

## Use

```sh
./wind            # print and exit
./wind --selftest # assert the pure helpers still behave
watch -n 300 ./wind
```

Python 3 stdlib only, no dependencies.

## Adding a spot

Edit `SPOTS` at the top. To find the ids for a windguru spot page
(`windguru.cz/<id_spot>`):

```sh
curl -s -H 'Referer: https://www.windguru.cz/' \
  'https://www.windguru.cz/int/iapi.php?q=spot&id_spot=<id_spot>' | python3 -m json.tool
```

`station.id_station` is the live station, if the spot has one. Spots without a
station print `no live data` and still show a forecast.
