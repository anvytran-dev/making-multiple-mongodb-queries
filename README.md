# Make Multiple MongoDB Queries in a Request Handler

![Project Image](/img/laptop.jpg)

After making a GET, POST, PUT, or DELETE request to our server --sometimes, we need our request handler to make MongoDB queries to different collections. 

## Why may we need to query multiple collections?

We may need to find data from different collections because:

We need to perform operations on data from two different collections
We need to return data so that we can use it when rendering our EJS file

Three ways that we can make multiple MongoDB queries in the same request handler are by 1) nesting callbacks, 2) utilizing Promise.all(), and utilizing the async...await keywords.

## Way #1: Nesting Callbacks

We can nest callbacks to make multiple MongoDB queries. 

In the following get request handler, we make 3 independent MongoDB queries by nesting callbacks. The callbacks run synchronously. 

First, Query #1 Runs 
Next, Query #2 Runs
Then, Query #3 Runs
```
app.get('/addTimeSlots', isLoggedIn, function (req, res) {

//Query #1:
 db.collection('signUpSheet').findOne({ _id: ObjectId(req.query.id) }, (err, eventDetails) => {

//Query #2
   db.collection('timeSlots').find({ eventId: req.query.id }).sort({date: 1}).toArray((error, slots) => {

//Query #3
     db.collection('userSignUp').find({}).toArray((error1, users) => {

  if (err) return console.log(err)

  res.render('addTimeSlots.ejs', {
   signUpSheet: eventDetails, //First Query's Result
   timeSlots: slots, //Second Query's Result
   userSignUp: users //Third Query's Result
  })
      })
    })
  })
});
```

Nesting callbacks allow us to make multiple MongoDB requests in the same request handler.

While nesting callbacks will certainly do the job of returning data from MongoDB, as the number of callbacks increase, our code can quickly become hard to read and it can be difficult to keep track of opening and closing brackets (callback hell). 

## Way #2: Promise.all()

Using Promise.all() to make multiple MongoDB queries is an alternative. 

First, the Promise.all() method takes an iterable of promises as its parameter.

Next, it returns a single Promise.

Then, when Promise.all() resolves, the single Promise resolves to an array of the results of the input promises. 

In this example, when using Promise.all(iterable) we will use an array as the iterable. 
```
app.get('/addTimeSlots', function (req, res) {

//Use variables to store queries for cleaner code

//Query #1
   let signUp = db.collection('signUpSheet').findOne({ _id: ObjectId(req.query.id) })

//Query #2
   let timeSlot = db.collection('timeSlots').find({ eventId: req.query.id }).sort({date: 1}).toArray()

//Query #3
   let guestSignUp = db.collection('userSignUp').find({}).toArray()

//The 3 queries are executed asynchronously.
//Promise.all() resolves when all 3 promises are resolved.
//After Promise.all() resolves, an array of the results are returned.

  Promise.all([signUp, timeSlot,guestSignUp]).then((returnedValues) => {

//returnedValues is the array of results we returned.
//We can deconstruct the array to make it easier to work with.

  const [signUpResults, timeSlotResults, guestSignUpResults] = 
  returnedValues;

    res.render('addTimeSlots.ejs', {
      signUpDetails: signUpResults, //OR returnedValues[0]
      timeSlotDetails: timeSlotResults, //OR returnedValues[1]
      guestSignUpDetails: guestSignUpResults //OR returnedValues[2]
     })
   }).catch((error) => {
     console.log(error)
   });
 });
 ```

In the above example, we can see that after Promise.all() resolves, we return an array containing the results of our 3 promises. After we receive the results, we can work with them.

In this example,  we used array deconstructing to make our code cleaner to work with. 

## Way #3: Async...await

The async...await syntax allows us greater readability in handling promises. It is concise, easy to use, and simple to understand. 

We must use the 'async' keyword before the function.  Next, we use the 'await' keyword before each promise(our queries in this case). The await keyword waits for the promise to resolve/reject.
```
//Use the 'async' keyword 
app.get('/addTimeSlots', async function (req, res) {

//Use variables to store queries for cleaner code
 
//Query #1, use the 'await' keyword
//After the await keyword, write the query. The query is now a promise.
 let signUpResults = await db.collection('signUpSheet').findOne({ _id: ObjectId(req.query.id) })
 
 //Query #2, use the 'await' keyword
 //After the await keyword, write the query. The query is now a promise.
 let timeSlotResults = await db.collection('timeSlots').find({ eventId: req.query.id }).sort({date: 1}).toArray()
 
 //Query #3, use the 'await' keyword
 //After the await keyword, write the query. The query is now a promise.
 let guestSignUpResults = await db.collection('userSignUp').find({}).toArray()

//Once the promises are resolved, the results will be returned.

 res.render('addTimeSlots.ejs', {
 signUpDetails: signUpResults, 
 timeSlotDetails: timeSlotResults, 
 guestSignUpDetails: guestSignUpResults 
        })
})
```
With this method, we can handle the queries asynchronously through using promises while having better readability. 

## Conclusion

Three ways that we can make multiple MongoDB queries in the same request handler are 1) nesting callbacks, 2) utilizing Promise.all(), and utilizing the async...await. While all ways return the data we need, using async...await is a cleaner and more efficient alternative to nesting callbacks or using Promise.all(). It allows us to keep track of our code easily while running asynchronously. 

## Resources
Callback Hell

Promise.all()

Array Deconstruction 

Async/Await
