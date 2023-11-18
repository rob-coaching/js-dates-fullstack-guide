# JavaScript Dates & Date Filtering

## Intro 
In JS there is a topic that s*cks like almost none other: DATE operations.

First of all some basics...

We can have a date repsented in different formats.

## Formats. Formats. Formats.

### Date as String

We can have the date represented as string with the day of the year in US format:

Example: 
`2023-07-11``

Straight forward. Next...

### Data as "ISO String"

A very common format to represent dates in a programming language is the "ISO String". 

It contains both: the day of the year + the time including milliseconds.

Example:

`2023-11-18T00:00:00.000Z`

The "T" in that string separates the date from the time. And the Z is the final delimiter.

This is the format that you will usually see when viewing your data in your MongoDB database
(e.g. in ATLAS or Compass).


### Date Object in JavaScript

For more flexibility to convert a date into different formats as we need or extract infos from it, we can store our date in a javascript DATE objects:

const myDate = new Date("2023-11-18")
console.log(myDate) // => `2023-11-18T00:00:00.000Z`

Notice that we get printed this ISO date string by default, when we console.log the date object.

But don't get fooled, little grasshopper. It is no string, ist is an OBJECT.

And therefore it provides us some nice stuff, like extracting parts of the date:


```
const myDate = new Date("2023-11-18")
console.log(myDate) // => `2023-11-18T00:00:00.000Z`
console.log(myDate.getDate()) // => 18 (DAY)
console.log(myDate.getFullYear()) // => 2023 (YEAR)
```

### Date as Number

There also exist a pure number format for a date. This is called a TIMESTAMP.

It is a date that represents the MILLISECONDS passed since a certain time.

Now the question is: The milliseconds passed since WHICH time?

Well, we just pick so date long enough ago.

The most prominent timestamp is the so called UNIX TIMESTAMP.

It is a date that represents the MILLISECONDS passend since the day <b>1970-01-01</b>

Examples:

```
2023-11-17T00:00:00.000Z (ISO DATE) => 1700179200000 (UNIX TIMESTAMP)
2023-11-18T00:00:00.000Z (ISO DATE) => 1700265600000 (UNIX TIMESTAMP)
```

Storing the date as such a number has some great advantages.

We can very easily COMPARE dates. 

Because we simply check if the numbers are equal, greater or less than the other. And so will be the dates.

This allows very efficient date comparisons & filterings, e.g. filtering all items between a certain date range.

The check will simply be if a date number is between two other date numbers. And then we know, this date is in between the date range.

This is a very common feature we need when we want to filter events or locations based on a start data & and end date, e.g. when booking a hotel or a flight.

Now: How to get this magical timestamp thingy? HOOOOOOW????

```
const myDate = new Date("2023-11-08")

console.log(myDate) // => `2023-11-18T00:00:00.000Z` (ISO DATE)

const myDateTimestamp = myDate.getTime() // get timestamp (= date as milliseconds)
console.log(myDateTimestamp) // => 1699401600000 (TIMESTAMP)

// check if date is in a range...
const myDateBefore = new Date("2023-11-01")
const myDateAfter = new Date("2023-11-20")
const myDateBeforeTimestamp = myDateBefore.getTime()
const myDateAfterTimestamp = myDateAfter.getTime()

// check if the date above is between these two other dates
if(myDateTimestamp >= myDateBeforeTimestamp && myDateTimestamp <= myDateAfterTimestamp ) {
  // this date is in between the other two ! do something....
}

```

## Date in HTML Input element

The HTML element `<input type="date" />` allows the user to pick a date from a datepicker.

How nice that is... right?

The RESULT of the picking will be - as usual - avaible in the event object:
`event.target.value`

The format of that picked date will usually be the ISO String format.

`2023-11-18T00:00:00.000Z`

So in case you send this date nowto you backend, the backend will receive this date here as a STRING.

That is important to keep in mind!

## Date Handling in Backend

Date strings are no different than any strings for the backend.

You will receive those either as PARAMS in a GET request.

Or you will receive those inside your BODY in a POST request.

Once you receive now the date ISO STRING in your backend, you probably ask yourself:

How do you store those dudes in MongoDB? 

### Date usage in MongoDB

In our Mongoose Schema we have the "Date" datatype:

Example:
```
const EventSchema = new Schema({
    title: String,
    dateStart: Date,
    dateEnd: Date
})

export const Event = model("Event", EventSchema)
```

This "Date" datatype is the JavaScript Date object.

But mongoose is flexible when inserting those dates:

You can either insert JavaScript data objects OR simply date strings:

E.g. the following should work fine:

```
// here two example (ISO) date strings we received from the frontend
const strDateStart = "2023-12-01T00:00:00.000Z"
const strDateEnd = "20223-12-31T00:00:00.000Z"
const title = "New crazy event in your hometown, buddy"

// construct new event object from data
const eventItem = { title, dateStart: strDateStart, dateEnd: strDateEnd }

// store in database
const event = await Event.create( eventItem )

```

And boooom. MongoDB will now store your item with the two dates.

But keep in mind: MongoDB will store those as full date objects including the TIME:

So if you look up your event in the Database it will look like this:

```
{
  title: 'New crazy event in your hometown, buddy',
  dateStart: 2023-12-01T00:00:00.000Z,
  dateEnd: 2023-12-31T00:00:00.000Z,
  _id: new ObjectId('6558493ef1fb848cdb013885')
}
```

### Date Filtering

Now let's move on to the exciting stuff.

Filtering all your events ON A CERTAIN DATE or IN BETWEEN TWO DATES.

#### Filtering events BETWEEN two dates

You can use the find() method on your model to query your data.

In this case we want to check, if our event starts AFTER the given start date.
And ends BEFORE the given end date.

For that we can use the MongoDB query operators "$gte" (=greater than or equals)
and "$lte" (=less than or equals)

```
const strDateStart = "2023-12-01"
const strDateEnd = "2023-12-31"

Event.find({
  dateStart: { $gte: strDateStart  },
  dateEnd: { $lte: strDateEnd }
})
```

If the frontend sends ISO date strings, this will work too.

#### Filtering events on ONE day

In case we just store the day of an event in the database and not the TIME, things are easy.


```
const strDateFilter = "2023-12-01T00:00:00.000Z"
// const strDateFilter = "2023-12-01" // this here will also work!

Event.find({ dateStart: strDateFilter })
```

But in case we also store additionally the TIME of the events in the database, this will unfortunately not work.

E.g. our event date is stored like that:

"2023-12-01T16:30:00.000Z"

This is where things get slightly tricky.

Now this date takes place on 16:30 o'clock.

Now we cannot simply check if that date equals the day:
"2023-12-01T00:00:00.000Z"

That will not work.

There is no Mongo builin method that we can use to only check just the DAY part.

But there is a nice workaround.

We can use again filtering BETWEEN two days.

In this case, if we want to figure out if an event is on a certain day, we will simply check:
Is the day BETWEEN the start of that day and the start of the NEXT day

Example:

Event date: "2023-12-01T16:30:00.000Z"

Start of day: "2023-12-01T00:00:00.000Z"
Start of next day: "2023-12-02T00:00:00.000Z"

So in case our event is between that two dates, we know, it happens on the first of december 2023.

So we have to do one thing in JavaScript:

We need to ADD ONE DAY to the received date.

And then we check all events if they are between the start date and the NEXT DAY date.

```
const strDate = "2023-12-01T00:00:00.000Z"
const dateStart = new Date(strDate) // convert to date
let dateNext = new Date(strDate) // copy data to second variable

// the mindfuck part....
dateNext.setDate( dateNext.getDate() + 1 )
dateNext = new Date( dateNext )

Event.find({
  dateStart: { $gte: dateStart  },
  dateEnd: { $lte: dateNext }
})

```

For the curious ones: 
What does that mindfuck part above - to calculate the next day - exactly do ?

```
dateNext.setDate( dateNext.getDate() + 1 )
dateNext = new Date( dateNext )
```

Step by step:
- getDate gets the DAY of the day => e.g. 2023-11-30 => 30
- adding +1 gets us the next day => 31
- setDate SETS / updates the day on the date => from 30 to 31
- setDate also checks if we reached the next month. e.g. when we are at day 31, it will go to day 01 and also increases the month! Same if we reached the end of the year
- but setDate converts the date to a TIMESTAMP (=> so a gibberish number like that 12034040404000)
- we need to convert this number back to a date object with "new Date( theNumber)"
- and there we go: Finally we got a date object for the next day 

And now we can compare if a date is between the given day and the next day:

```
Event.find({
  dateStart: { $gte: date  },
  dateEnd: { $lte: dateNext }
})
```

And that's it.

Amen.


# OUTRO

Enjoy dating around, little grasshopper....

