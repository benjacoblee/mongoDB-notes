# MongoDB Notes

MongoDB is a "schema-less" data solution. Stores documents in JSON format, saved as BSON on the server (binary).

### Basic Shell Commands

    show dbs (shows all databases)
    use products // connects to DB, or implicitly creates one after document insertion
    db.products.insertOne({name: "A T-shirt"}) 
    db.products.find() // returns all products since no filter

### CRUD Operations

###### Create

    db.products.insertOne(
      {
      name: "A T-shirt",
      price: 12.99
      })

    db.products.insertMany([
      { // takes an array
      name: "A T-shirt",
      price: 12.99
      }, 
      {
      name: "A computer",
      price: 1299,
      details: "Very good computer"
    }])

###### Read

    db.products.find({
      name: "A T-shirt" // filter. Finds all entries with name === "A T-shirt"
    })

The filter can take in multiple arguments:

    db.products.find({
      name: "A T-shirt",
      price: 12.99 // look for entries where name === "A T-shirt" && price === 12.99
    })

    db.products.findOne({
      name: "A T-shirt"
    }) // .findOne returns the FIRST entry it finds, regardless if there are other entries that fulfill the requirements

###### Update

    db.products.updateOne({ // updates the first document it finds
      name: "A T-shirt" // first argument is the filter
    }, {
      $set: { // second argument
        name: "A really nice t-shirt" // set the name field 
      }  
    })

We can add key-value pairs that didn't exist previously. Assuming there was no status field:

    db.products.updateOne({
      name: "A T-shirt",
    }, {
      $set: {
        name: "A really nice t-shirt",
        status: { // add this field
          availability: 5 
        }
      }
    })

We can update many products:

    db.products.updateMany({
      name: "A really nice t-shirt" // filter
    }, {
      $set: {
        name: "A T-shirt"
      }
    })

We can replace a product altogether:

    db.products.replaceOne({
      name: "A T-shirt"
    }, {}) // second argument being what we want to replace the existing document with. In this case, replaces entry with an empty object instead

###### Delete

    db.products.deleteOne({
      name: "A T-shirt" // deletes the first entry with name === "A T-shirt"
    })

    db.products.deleteMany({}) // if no filter is provided, deletes everything

###### Query and Projection Operators

We can use projection operators to narrow down our search. For example, perhaps we want to find products that are >= $1. We can do that with projection operators:

    db.products.find({price: {$gte: 1}})

If we want to omit certain fields for optimization:

    db.products.find({name:"A T-shirt"}, {name: 1}) // Returns only objectID and name

    db.products.find({name:"A T-shirt"}, {name: 1, _id: 0}) // Returns name only

###### Accessing Embedded Documents

Say, for example, we had a document that looked like this:

     "_id" : ObjectId("5e9680e8a266d47f98d63acb"),
        "name" : "A T-shirt",
        "price" : 12.99,
        "colors" : [
                "blue",
                "black",
                "white"
        ],
        "details" : {
                "description" : "Really good quality"
        }

We can access the array like this: 

    db.products.findOne({name:"A T-shirt"}).colors

Or maybe we want to find all T-shirts with black color:

    db.products.find({colors:"black"})

If we want to find a product with description field === "Really good quality":

    db.products.find({"details.description": "Really good quality"})
    // Note: If we use dot notation, we must wrap the key in double quotes!

### How to structure data

- Schemas
- Relations
- Validations

### Why use schemas?

We're not forced by MongoDB to adhere to a specific schema, but having data structured a certain way makes rendering a view easier (given we know how the data is supposed to look, what data we can use).

### Data Types - An Overview
    Text => "Ben"
    Booleans => true/false
    Numbers => int32 (integer), int64 (numberLong), numberDecimal
    ObjectIds
    Dates => ISODate, Timestamp
    Embedded documents => objects
    Arrays

### Data Structures & Data Modelling

Some considerations:

- What data does my app generate or need? (User info, orders)
- Where do I need my data? (Welcome page, products list page)
- Which kind of data do I want to display i.e. what queries do I need to write?
- How often do I fetch the data? Do I need to optimize?
- How often do I need to read/write data?

### Relations Options

###### Nested/Embedded Documents:

    {
      userName: "Ben",
      age: 30,
      address: {
        street: "Some Street 11,
        city: "Singapore
      }
    }

###### References: 

    {
      userName: "Ben",
      faveBooks: [{
        id: 1,
        name: "Blink"
      }, {
        id: 2,
        name: "Outliers"
      }]
    }

###### OR:

    {
      userName: "Ben",
      faveBooks: ["id1", "id2"]
    }

    // Then in books collection:

    {
      _id: "id1",
      name: "Blink"
    }

###### An example: 

    db.diseaseSummaries.insertOne({
      _id: 1,
      diseases: ["sinus", "torn ACL"]
    })

    db.patients.insertOne({
      name: "Ben",
      age: 30,
      diseaseSummaryID: 1
    })

    let benDiseaseSummaryID = db.patients.findOne({name:"Ben"}).diseaseSummaryID

    db.diseaseSummaries.findOne({_id: benDiseaseSummaryID})

    // returns { "_id" : 1, "diseases" : [ "sinus", "torn ACL" ] }

HOWEVER. This is a 1-1 relationship, meaning one patient will always only have one summary, and one summary will only belong to one patient, embedding might be a better option.

    db.patients.insertOne({
      name: "Ben",
      age: 30,
      diseaseSummary: {
        diseases: ["sinus", "torn ACL"]
      }
    })



