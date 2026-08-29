# wind

Terminal wind scraper for [windguru.cz](https://www.windguru.cz) — live station
readings plus GFS forecast for Bat Galim (Haifa) and Caesarea.

```
$ wind

  Bat Galim   7.6 kt g11.9  ↗ SW 219°  31°C  09:52
  Caesarea   14.2 kt g17.1  ↑ S  198°  28°C  09:53
  Tel Aviv    5.4 kt  g9.3  ↖ SE 154°  26°C  09:54
  Ashdod      5.4 kt  g8.2  ↖ SE 148°  26°C  09:54

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
./wind                  # Bat Galim + Caesarea, print and exit
./wind -a               # all four spots
./wind tel --watch 15   # Tel Aviv, live, repaint every 15s
./wind --watch          # the default two, live, 30s
./wind --alert          # Telegram anything at or above 15 kt, then exit
./wind -a --alert 18    # ...all spots, 18 kt
./wind --help
./wind --selftest       # assert the pure helpers still behave
```

Spot arguments are case-insensitive substrings of the label, so `bat`, `Bat`
and `galim` all pick Bat Galim. With no arguments you get the `DEFAULT` list at
the top of the script; `-a` overrides it.

`--watch` quits on `q`, Esc or Ctrl-C. Piped or run without a tty it just loops
and prints, so `./wind --watch 60 >> wind.log` works too.

Python 3 stdlib only, no dependencies.

The forecast is cached in `$TMPDIR` for 15 minutes, so a 30s repaint only
re-hits the live station. Delete `/tmp/wind-fcst-*.json` to force a refetch.

## Adding a spot

Edit `SPOTS` at the top. To find the ids for a windguru spot page
(`windguru.cz/<id_spot>`):

```sh
curl -s -H 'Referer: https://www.windguru.cz/' \
  'https://www.windguru.cz/int/iapi.php?q=spot&id_spot=<id_spot>' | python3 -m json.tool
```

`station.id_station` is the live station, if the spot has one. Use `None` for a
spot with no station — it prints `no live data` and still shows a forecast.

Not every named spot has one. Of the Israeli spots checked: Bat Galim 2049,
Caesarea/Freegull 2259, Tel Aviv ims 3169, Ashdod port ims 3444 all report;
plain `Tel-Aviv` (308), `ashdod` (95740) and most others are forecast-only.

## Telegram alerts

`--alert [KT]` checks the live stations once, sends a Telegram message for any
spot at or above the threshold (default 15 kt, avg **or** gust), and exits. It
prints what it sent, so it is safe to run from cron:

```cron
*/10 6-20 * * * /home/you/github/wind/wind -a --alert >> /tmp/wind-alert.log 2>&1
```

Credentials come from the environment, or from `~/.config/wind/telegram.env`:

```sh
mkdir -p ~/.config/wind
cat > ~/.config/wind/telegram.env <<'EOF'
WIND_TG_TOKEN=123456:AA...      # @BotFather -> /token
WIND_TG_CHAT=-1001234567890     # or @publicchannelname
EOF
chmod 600 ~/.config/wind/telegram.env
```

Add the bot to the channel **as an admin**, or Telegram rejects the send. Then:

```sh
./wind --tg-chats   # lists every chat the bot can see, with its id
./wind --tg-test    # sends one message, to prove the wiring
```

`--tg-chats` reads `getUpdates`, and Telegram only reports chats with recent
activity — so post something in the channel first. Note that a bot with a
webhook set, or one whose updates another process is already consuming, will
show nothing here; grab the id from the channel's forwarded-message info
instead.

To stop it nagging: one alert per spot, then silence for 3 hours
(`ALERT_COOLDOWN`). If the wind drops back under the threshold the spot re-arms
immediately, so a new build gets its own message. State lives in
`~/.cache/wind-alerts.json` — delete it to reset.
