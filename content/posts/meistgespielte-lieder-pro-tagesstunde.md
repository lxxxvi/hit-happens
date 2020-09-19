---
title: "SRF3: Meistgespielte Lieder Pro Tagesstunde"
description: "Welche Lieder werden in welcher Tagesstunden am ehesten gespielt?"
date: 2020-09-18T16:16:40+02:00
draft: false
---

* **Station: SRF3**
* **Zeitspanne: 1. Januar 2020 - 17. September 2020**

Bei dieser Auswertung ging es darum, für jede Tagesstunde den meistgespielten Song zu finden.

## Beobachtungen

2 Lieder gewinnen eine Tagesstunde gleich 2 Mal:

- _Luca Hänni - SHE GOT ME_ zwischen _00:00 - 00:59_ und _01:00 - 01:59_.
- _PINK/CASH CASH - CAN WE PRETEND_ zwischen _05:00 - 05:59_ und _17:00 - 17:59_.

Die Stunde _20:00 - 20:59_ scheint am abwechslungsreichsten zu sein: Der Song _Kt Gorique - AIRFORCE_ ist Sieger in dieser Stunde mit gerade mal 4 Plays während dieser Tagesstunde.

Das Lied _BLUE WEDNESDAY - BIRTHDAY GIRL_ welches die Tagesstunde _10:00 - 10:59_ gewinnt, wird praktisch jeden Sonntag um die gleiche Zeit (ca. 10:58) angespielt, quasi Tradition.

_The Weeknd - BLINDING LIGHTS_ gewinnt Tagesstunde _15:00 - 15:59_, dies aufgrund der wöchentlichen Hitparade am Sonntag. [Dieser Post](/hit-happens/posts/weeknd-blinding-lights-plays-in-tagesstunde/) erklärt das Phänomen genauer.

## Resultat

| Tagesstunde | Plays | Künstler - Song |
|:-|:-:|:-|
| 00:00 - 00:59 | _51_ | **Luca Hänni - SHE GOT ME** |
| 01:00 - 01:59 | _38_ | **Luca Hänni - SHE GOT ME** |
| 02:00 - 02:59 | _38_ | Marc Sway - BEAT OF MY HEART |
| 03:00 - 03:59 | _38_ | Robert Miles - Children |
| 04:00 - 04:59 | _20_ | DA SIGN AND OPPOSITE - SLOW DOWN TAKE IT EASY |
|               | _20_ | DANITSA - CAPTAIN |
| 05:00 - 05:59 | _14_ | JAX JONES FEAT. BEBE REXHA - HARDER |
|               | _14_ | **PINK/CASH CASH - CAN WE PRETEND** |
| 06:00 - 06:59 | _15_ | RITON/OLIVER HELDENS FEAT. VULA - TURN ME ON |
| 07:00 - 07:59 | _13_ | Nothing But Thieves - IS EVERYBODY GOING CRAZY? |
|               | _13_ | SZA/JUSTIN TIMBERLAKE - THE OTHER SIDE |
| 08:00 - 08:59 | _13_ | Jain - ALRIGHT |
|               | _13_ | Twenty One Pilots - LEVEL OF CONCERN |
| 09:00 - 09:59 | _14_ | CALVIN HARRIS/RAG'N'BONE MAN - GIANT |
|               | _14_ | SAINT JHN - ROSES |
| 10:00 - 10:59 | _36_ | BLUE WEDNESDAY - BIRTHDAY GIRL |
| 11:00 - 11:59 | _14_ | FREYA RIDINGS - CASTLES |
|               | _14_ | Post Malone - CIRCLES |
| 12:00 - 12:59 | _13_ | RAY DALTON - IN MY BONES |
| 13:00 - 13:59 | _14_ | Lewis Capaldi - BEFORE YOU GO |
|               | _14_ | MARK RONSON FEAT. MILEY CYRUS - NOTHING BREAKS LIKE A HEART |
|               | _14_ | MEDUZA/BECKY HILL/GOODBOYS - LOSE CONTROL |
|               | _14_ | REGARD - RIDE IT |
| 14:00 - 14:59 | _19_ | KAROL G/NICKI MINAJ - TUSA |
| 15:00 - 15:59 | _35_ | The Weeknd - BLINDING LIGHTS |
| 16:00 - 16:59 | _14_ | RITON/OLIVER HELDENS FEAT. VULA - TURN ME ON |
| 17:00 - 17:59 | _12_ | **PINK/CASH CASH - CAN WE PRETEND** |
| 18:00 - 18:59 | _14_ | Camila Cabello - LIAR |
| 18:00 - 18:59 | _14_ | Katy Perry - NEVER REALLY OVER |
| 19:00 - 19:59 | _19_ | KUSH K - FOREVER ONLY |
| 20:00 - 20:59 | _4_  | Kt Gorique - AIRFORCE |
| 21:00 - 21:59 | _8_  | Alois - AFTER LIFE |
| 22:00 - 22:59 | _8_  | Phoebe Bridgers - KYOTO |
| 23:00 - 23:59 | _11_ | JOHN BÜRGIN - CH BEATS DJ SET: JOHN BÜRGIN |

## Diagramm

### Gewinner der ersten Tageshälfte (00:00 - 11:59)

![Diagram der Gewinner der ersten Tageshälfte](morning_songs.jpg)

### Gewinner der zweiten Tageshälfte (12:00 - 23:59)

![Diagram der Gewinner der zweiten Tageshälfte](evening_songs.jpg)

SQL Query für die Datenaufbereitung:

```sql
SELECT hour
     , plays
     , a.name || ' - ' || s.title AS artist_song
  FROM (
  SELECT hour
       , song_id
       , RANK() OVER (PARTITION BY hour ORDER BY plays DESC)
       , plays
    FROM (
    SELECT song_id
         , EXTRACT(hour FROM broadcasted_at_in_tz) AS hour
         , count(*) AS plays
      FROM (
      SELECT song_id
           , broadcasted_at AT TIME ZONE 'UTC' AT TIME ZONE 'Europe/Zurich' broadcasted_at_in_tz
        FROM cleaned_broadcasts
     ) t1
     GROUP BY song_id, hour
  ) t2
) t3
 INNER JOIN songs s ON s.id = t3.song_id
 INNER JOIN artists a ON a.id = s.artist_id
 WHERE rank = 1
 ORDER BY hour, a.name
```
