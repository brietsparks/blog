---
title: Three Exercises for Teaching JavaScript Arrays and Loops
date: "2020-09-21T10:00:00.000Z"
---
Here are a few exercises I wrote for JavaScript bootcamp students who had just learned array and loop basics. Feel free to use! Each exercise has a goal and prompt. The goal tells the teacher the intended outcome of the exercise, and the prompt is what the student reads.

## Exercise 1
### Goal: 
Given an array of objects, for each object, `console.log` a string derived from two or more attributes. The student should demonstrate use of a for-loop or foreach-method.

### Prompt:
Loop through the following array to `console.log` a sentence describing the weather of each day:

```js
const weeklyForecast = [
  { day: 'monday', hi: 90, lo: 70 },
  { day: 'tuesday', hi: 93, lo: 76 },
  { day: 'wednesday', hi: 89, lo: 74 },
  { day: 'thursday', hi: 91, lo: 76 },
  { day: 'friday', hi: 82, lo: 71 },
  { day: 'saturday', hi: 81, lo: 68 },
  { day: 'sunday', hi: 86, lo: 64 },
];
```

For example, each sentence would look like:

`On Monday, the high will be 90 degrees and the low will be 70 degrees.`



## Exercise 2
### Goal: 
Given an array of objects, obtain an array that contains a subset of the data that has been filtered by criteria. The student should demonstrate use of a for-loop or filter-method. Note, this exercise assumes no knowledge of working with date objects, so the dates are strings. 

### Prompt:
CapitalTwo has an online banking website. A user’s account transaction history looks like:

```js
const originalTransations = [
  { transactionId: 't100', type: 'debit', amount: 107.15, description: 'Amazon Purchase', date: '09/01/2020' },
  { transactionId: 't101', type: 'debit', amount: 15.05, description: 'QuikTrip', date: '09/01/2020' },
  { transactionId: 't102', type: 'debit', amount: 9.67, description: 'Chipotle', date: '09/02/2020' },
  { transactionId: 't103', type: 'debit', amount: 350, description: 'A1 Air Conditioning', date: '09/03/2020' },
  { transactionId: 't104', type: 'debit', amount: 12.30, description: 'Chick Fil A', date: '09/03/2020' },
  { transactionId: 't105', type: 'credit', amount: 500, description: 'Deposit', date: '09/05/2020' },
  { transactionId: 't106', type: 'debit', amount: 25, description: 'DPS Service Fee', date: '09/06/2020' },
  { transactionId: 't107', type: 'debit', amount: 212.31, description: 'Sprouts', date: '09/06/2020' },
  { transactionId: 't107', type: 'credit', amount: 20.90, description: 'Sprouts', date: '09/06/2020' },
  { transactionId: 't108', type: 'debit', amount: 11.50, description: 'Half Price Books', date: '09/07/2020' }
]
```

A user needs to be able to filter their transactions by the purchase amount. For example, filtering purchases that are greater than 200 should give us:

```js
[
  { transactionId: 't103', type: 'debit', amount: 350, description: 'A1 Air Conditioning', date: '09/03/2020' },
  { transactionId: 't107', type: 'debit', amount: 212.31, description: 'Sprouts', date: '09/06/2020' }
]
```

In this exercise, show how you can loop through the `originalTransactions` to obtain a new array for each of the following criteria:

- purchases less than or equal to 20
- transactions that are whole-dollar amounts
- transactions that are credits or occurred on 9/3/2020


## Exercise 3
### Goal
Given a paragraph of text, return an object containing the count of occurrences of each string of an array of strings. The student should demonstrate use of a for-loop or reduce-method.

### Prompt:
RevolvingDoor is a government job board that needs to be able to aggregate key word occurrences in candidate resumes. A resume might look like:

```
Kyle the Coder

Infinite Loop Inc., 2017 - 2019
- wrote migration scripts for SQL databases
- built reusable UI components with React

Varlet Agency, 2016 - 2017
- created user interfaces using JS, React, and CSS
- built API services with NodeJS and MongoDB

InfoSysDigiTechSphere Consulting Solutions, 2015 - 2016
- built API services with in Java, Spring Boot, and SQL
```

Suppose an employer wants to see the keyword counts of "SQL", "API", and "migration". It would look like:

```js
const keywordOccurrences = {
  SQL: 2,
  API: 2,
  migration: 1
}
```

In this exercise, show how you can loop through the words of the resume and aggregate the counts of the following keywords: "React", "UI", "NodeJS", "Java", "Spring", "MongoDB". Hint: use the string `.split` method to get started.
