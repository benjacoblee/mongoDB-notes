# MongoDB Notes

MongoDB is a "schema-less" data solution. Stores documents in JSON format, saved as BSON on the server (binary).

###### Basic Shell Commands

    show dbs (shows all databases)
    use products // a currently connected DB, or implicitly creates one after document insertion
    db.products.insertOne({name: "A T-shirt"}) 
    db.products.find() // returns all products since no filter

###### CRUD Operations

CREATE

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

READ

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

UPDATE

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

DELETE

    db.products.deleteOne({
      name: "A T-shirt" // deletes the first entry with name === "A T-shirt"
    })

    db.products.deleteMany({}) // if no filter is provided, deletes everything

QUERY AND PROJECTION OPERATORS

We can use projection operators to narrow down our search. For example, perhaps we want to find products that are >= $1. We can do that with projection operators:

    db.products.find({price: {$gte: 1}})

If we want to omit certain fields for optimization:

    db.products.find({name:"A T-shirt"}, {name: 1}) // Returns only objectID and name

    db.products.find({name:"A T-shirt"}, {name: 1, _id: 0}) // Returns name only

ACCESSING EMBEDDED DOCUMENTS

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



