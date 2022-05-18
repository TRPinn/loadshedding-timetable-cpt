# Loadshedding Timetable - CPT

Here is the [Cape Town loadshedding timetable](https://www.capetown.gov.za/Loadshedding1/loadshedding/maps/Load_Shedding_All_Areas_Schedule_and_Map.pdf) in JSON format. I hoped this to be a convenient format for running queries on or transforming.

I've included sample code (C#) below:
- Models used in the timetable
- Read and deserialize file
- Query timetable using LINQ

## Models

```csharp
class Event
{
    public int Day { get; set; }
    public List<Timeslot> Timeslots { get; set; } = new List<Timeslot>();
}

class Timeslot
{
    public TimeSpan Start { get; set; }
    public TimeSpan End { get; set; }
    public Dictionary<int, int[]> StageToAreas { get; set; } = new Dictionary<int, int[]>();
}
```

## Read and deserialize file

```csharp
using Newtonsoft.Json;
var contents = File.ReadAllText("timetable.json");
var timetable = JsonConvert.DeserializeObject<List<Event>>(contents);
```

## Query timetable using LINQ

The query below can be used to find upcoming loashedding timeslots. 

```csharp
var day = 1; // min=1,max=31
var stage = 1; // min=1,max=8
var area = 1; // min=1,max=16
var hours = 12; // min=0,max=23
var minutes = 30; // min=0,max=59

var slots = timetable.First(evt => evt.Day == day)
    .Timeslots.Where(timeslot =>
        timeslot.Start >= new TimeSpan(hours, minutes, 0) &&
        timeslot.StageToAreas.Any(stageToArea =>
            stageToArea.Key == stage &&
            stageToArea.Value.Contains(area)))
    .ToList();
```