# TimesDates
### Nanosecond Resolvable Times with Dates, or Times with Dates in TimeZones.
#### Released under the MIT License. Copyright &copy; 2018 by Jeffrey Sarnoff.

> _this package requires Julia v0.7-_

- the type `TimeDate` works with DateTime, Date & Time (Dates.jl)

- the type `TimeDateZone` works with ZonedDateTime, TimeZone (TimeZones.jl)

----

### [Setup](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#setup)

### [`TimeDate`](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#timedate-is-nanosecond-resolved)

### [`TimeDateZone`](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#timedatezone-is-nanosecond-resolved-and-zone-situated)

### [Examples](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#additional-examples)

### [The Design](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#the-design)

### [Acknowledgments](https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#acknowledgements)

----

## Setup

```julia
using Pkg3
add TimesDates
```
or
```julia
using Pkg
add("TimesDates")
```

## In Use

#### `TimeDate` is nanosecond resolved

```julia
julia> using TimesDates, Dates

julia> td2018 = TimeDate("2018-01-01T00:00:00.000000001")
2018-01-01T00:00:00.000000001

julia> td2017 = TimeDate("2018-01-01T00:00:00") - Nanosecond(1)
2017-12-31T23:59:59.999999999

julia> td2017 < td2018
true

julia> td2018 - td2018
2 nanoseconds
```

----

#### `TimeDateZone` is nanosecond resolved and zone situated

```julia
julia> using Dates, TimeZones, TimesDates

julia> zdt = ZonedDateTime(DateTime(2012,1,21,15,25,45), tz"America/Chicago")
2012-01-21T15:25:45-06:00
https://github.com/JeffreySarnoff/TimesDates.jl/blob/master/README.md#additional-examples
julia> tdz = TimeDateZone(zdt)
2012-01-21T21:25:45 America/Chicago

julia> tdz += Nanosecond(123456)
2012-01-21T21:25:45.000123456 America/Chicago

julia> ZonedDateTime(tdz)
2012-01-21T21:25:45-06:00
```

### Additional Examples

Get components.
```julia
julia> timedate = TimeDate("2018-03-09T18:29:34.04296875")
2018-03-09T18:29:34.04296875

julia> Month(timedate), Microsecond(timedate)
(3 months, 968 microseconds)

julia> month(timedate), microsecond(timedate)
(3, 968)
```
Interconvert.
```julia
julia> date = Date("2011-02-05")
2011-02-05

julia> timedate = TimeDate(date); timedate, Date(timedate)
2011-02-05T00:00:00, 2011-02-05

julia> datetime = DateTime("2011-02-05T11:22:33")
2011-02-05T11:22:33

julia> timedate = TimeDate(datetime); timedate, DateTime(timedate)
2011-02-05T11:22:33, 2011-02-05T11:22:33
```
Get and Set the timezone to be used when no zone is specified.
Without this setting, UTC is used as the default.
```julia
julia> tzdefault()
UTC

julia> tzdefault!(localzone()); tzdefault()
America/New_York (UTC-5/UTC-4)

julia> tzdefault!(tz"Europe/London"); tzdefault()
Europe/London (UTC+0/UTC+1)
```
Use `localtime` and `uttime`.
```julia
julia> using Dates, TimeZones, TimesDates

julia> tzdefault!(localzone())
America/New_York (UTC-5/UTC-4)

julia> localtime()
2018-03-14T16:41:17.249 America/New_York

julia> uttime()
2018-03-14T20:41:17.251 UTC

julia> localtime(TimeDate), localtime(TimeDateZone)
(2018-03-14T16:41:17.252, 2018-03-14T16:41:17.252 America/New_York)

julia> uttime(TimeDate), uttime(TimeDateZone)
(2018-03-14T20:41:17.254, 2018-03-14T20:41:17.254 UTC)

julia> dtm = now() - Month(7) - Minute(55)
2017-08-14T15:46:17.256

julia> localtime(dtm)
2017-08-14T15:46:17.256 America/New_York

julia> uttime(dtm)
2017-08-14T19:46:17.256 UTC
```


## The Design

`Dates` has a `Time` type that has nanosecond resolution; it is not well supported, even within `Dates`.  This `Time` type recognizes strings only if they are limited to millisecond resolution.  It does form strings with nanosecond resolution.  This situation is exacerbated with the `DateTime` type.  Only millisecond (or coarser) resolved times are `DateTime` useful.

`TimeZones` is a laudable package.  At present it is limited by the millisecond barrier that accompanies `DateTime`.

This package exists to provide the community with two types.  One type `TimeDate` holds the caledar date with the time of day in nanoseconds.  The other `TimeDateZone` holds the the caledar date with the time of day in nanoseconds and the timezone.

The inner dynamics rely upon the `Period` types (`Year` .. `Day`, `Hour`, .., `Nanosecond`) and `CompoundPeriod` all provided by `Dates`.  We distinguish _slowtime_, which is millisecond resolved, from a nanosecond resolved _fasttime_.

```julia
julia> using Dates, TimesDates

julia> datetime = now()
2018-03-15T06:41:33.643

julia> currentdate = Date(datetime)
2018-03-15

julia> currenttime = Time(datetime)
06:41:33.643

julia> highrestime = currenttime + Nanosecond(98765)
06:41:33.643098765

julia> date = currentdate
2018-03-15

julia> fasttime = Microsecond(highrestime) + Nanosecond(highrestime)
98 microseconds, 765 nanoseconds

julia> slowtime = highrestime - fasttime
06:41:33.643
```

The general approach is separate the date, slowtime, fasttime, and timezone (if appropriate), then use the date, slowtime and timezone (if appropriate) to obtain a coarse result using the facilities provided by the `Date` and `TimeZones` packages.  We refine the coarse result by adding or subtracting the fasttime, as appropriate.

tbd [About TimesDates](https://jeffreysarnoff.github.io/TimesDates.jl/)

## Acknowledgements

This work is built atop `Dates` and `TimeZones`.    
While both are collaborative works, a few people deserve mention:
- `Dates`: Jacob Quinn and Stefan Karpinski
- `TimeZones`: Curtis Vogt

## Comments and Pull Requests are welcomed

http://github.com/JeffreySarnoff/TimesDates.jl
