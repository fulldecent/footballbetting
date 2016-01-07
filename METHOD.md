GET DATA AND SETUP
==================

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
cd nfllines cat nflcsv | grep -v Date | sed -e 's|([0-9])/([0-9])/([0-9])|\3-\1-\2|' > 1978to2009.csv
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
sqlite3 nfl.db CREATE TABLE games ( day DATE, vteam VARCHAR(50), vscore INTEGER, hteam VARCHAR(50), hscore INTEGER, line INTEGER, totalline INTEGER ); .separator "," .import Lines/nfllines/1978to2009.csv games create index gamesday on games (day); create index gamesvteam on games (vteam); create index gameshteam on games (hteam);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
CREATE TEMPORARY TABLE ab AS SELECT * FROM games UNION SELECT day, hteam, hscore, vteam, vscore, -line, totalline FROM games; # Make two copies of each game, with either team being a or b create index abbey on ab (day); create index abvteam on ab (vteam); create index abhteam on ab (hteam);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BET ON VISITOR?
---------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( select vscore + line - hscore as a from games ) group by a/abs(a) order by a desc; # 1 3603 # bet wins # 203 # bet ties # -1 3600 # bet loses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BET ON UNDERDOG?
----------------

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( SELECT CASE WHEN line>0 THEN (vscore+line)-hscore ELSE hscore-(vscore+line) END as a FROM games ) group by a/abs(a) order by a desc; # 1 3735
# 203
# -1 3468
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BET ON 14-DAY TRANSITIVES?
--------------------------

### Bet on A over B after A won X more points against C than did B.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select g1.day, g2.day, g3.day, g1.vteam, g1.hteam, g2.hteam, (g1.vscore+g1.line)-g1.hscore as Y, ((g2.vscore-g2.hscore) - (g3.vscore-g3.hscore)) as X from games g1, ab g2, ab g3 where g1.day > g2.day AND g1.day > g3.day AND g1.day < DATE(g2.day, '+14 days') AND g1.day < DATE(g3.day, '+14 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g3.vteam) AND (g2.hteam = g3.hteam);
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Bet on A over B after A beat C and C beat B. (Beat defined by game score)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( select (g1.vscore+g1.line)-g1.hscore as a from games g1, ab g2, ab g3 where g1.day > g2.day AND g1.day > g3.day AND g1.day < DATE(g2.day, '+14 days') AND g1.day < DATE(g3.day, '+14 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g3.vteam) AND (g2.hteam = g3.hteam) AND (g2.vscore > g2.hscore AND g3.vscore < g3.hscore) UNION select g1.hscore-(g1.vscore+g1.line) from games g1, ab g2, ab g3 where g1.day > g2.day AND g1.day > g3.day AND g1.day < DATE(g2.day, '+14 days') AND g1.day < DATE(g3.day, '+14 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g3.vteam) AND (g2.hteam = g3.hteam) AND (g2.vscore < g2.hscore AND g3.vscore > g3.hscore) ) group by a/abs(a) order by a desc; # 1 12 # 1
# -1 13
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Bet on A over B after A beat C and C beat B. (Beat defined by the line)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( select (g1.vscore+g1.line)-g1.hscore as a from games g1, ab g2, ab g3 where g1.day > g2.day AND g1.day > g3.day AND g1.day < DATE(g2.day, '+14 days') AND g1.day < DATE(g3.day, '+14 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g3.vteam) AND (g2.hteam = g3.hteam) AND (g2.vscore+g2.line > g2.hscore AND g3.vscore+g3.line < g3.hscore) UNION select g1.hscore-(g1.vscore+g1.line) from games g1, ab g2, ab g3 where g1.day > g2.day AND g1.day > g3.day AND g1.day < DATE(g2.day, '+14 days') AND g1.day < DATE(g3.day, '+14 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g3.vteam) AND (g2.hteam = g3.hteam) AND (g2.vscore+g2.line < g2.hscore AND g3.vscore+g3.line > g3.hscore) ) group by a/abs(a) order by a desc; # 1 8 # 1
# -1 15
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

BET ON 21-DAY PAYBACK?
----------------------

### Bet on A over B after B just beat A. (Beat defined by game score)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( select (g1.vscore+g1.line)-g1.hscore as a from games g1, ab g2 where g1.day > g2.day AND g1.day < DATE(g2.day, '+21 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g2.hteam) AND (g2.vscore < g2.hscore) UNION select g1.hscore-(g1.vscore+g1.line) as a from games g1, ab g2 where g1.day > g2.day AND g1.day < DATE(g2.day, '+21 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g2.hteam) AND (g2.vscore > g2.hscore) ) group by a/abs(a) order by a desc; # 1 30 # 1
# -1 30
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Bet on A over B after B just beat A. (Beat defined by the line)

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
select a/abs(a), count() from ( select (g1.vscore+g1.line)-g1.hscore as a from games g1, ab g2 where g1.day > g2.day AND g1.day < DATE(g2.day, '+21 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g2.hteam) AND (g2.vscore+g2.line < g2.hscore) UNION select g1.hscore-(g1.vscore+g1.line) as a from games g1, ab g2 where g1.day > g2.day AND g1.day < DATE(g2.day, '+21 days') AND (g1.vteam = g2.vteam) AND (g1.hteam = g2.hteam) AND (g2.vscore+g2.line > g2.hscore) ) group by a/abs(a) order by a desc; # 1 27 # 1
# -1 29
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
