This document provides a tutorial on storing data in a web browser by using IndexedDB. The lesson is short, focused, and has no dependencies on external libraries. The document assumes that you have some experience with modern web development.

For a more complete tutorial, see the IndexedDB article in The Modern Javascript Tutorial. For documentation about IndexedDB, see the IndexedDB API page at MDN.

You can find the code for this tutorial on GitHub.

# Getting Started with IndexedDB
IndexedDB is the most powerful and flexible way to store data in modern web browsers. Cookies are optimal for storing short strings, whereas IndexedDB can store JSON objects. LocalStorage is a good choice for storing JSON objects, but LocalStorage blocks the main execution thread, and offers no organization beyond simple key-value pairs.

IndexedDB is a complete, transactional, NoSQL database that enables you to store JSON objects in multiple tables. Unlike a relational (SQL) database, you cannot query IndexedDB. IndexedDB provides sequential access to records according to an index of records. 
## Core concepts
The web browser can store multiple IndexedDB databases per origin. Each database contains multiple tables, known as Object Stores. An object store contains a key and a collection of field values for each record. As a developer, you can define one or more indexes over any field in your record schema.

<drawing>

IndexedDB is an asynchronous process in the browser. Each interaction with the database follows a pattern of making a request, and providing callback methods to process the result of the request.

Every request that you make to the database must execute in the context of a transaction. The first step in every interaction is to request a Transaction object from the database, then request an ObjectStore object from the transaction. You perform most operations (create, retrieve, update, delete) on the ObjectStore that you retrieve from a Transaction.

<drawing>

## Example
The following example is a simple, one-page application. The application provides buttons to open the database, add (pre-defined) records, display records, and delete one record at random. The results are displayed in a `div` element on the page.


### Create the user interface: Scaffolding
Before you begin: create an empty file in your editor.

1. In an empty file, in your text editor, add an HTML element.

```
<html>
</html>
```

2. Inside your HTML element, add four `button` elements. Give each element a unique id attribute. The buttons enable you to interact with the database.

```
<html>
	<button id="openDB">Open DB object</button>
	<button id="addData">Add data</button>
	<button id="showData">Show Data</button>
	<button id="deleteData">Delete random row</button>
</html>
```

3. Below the `button` elements, add a `div` element. Give the `div` element a unique `id` attribute. You can use the `div` element to display the status of your database operations.

```
<html>
	<button id="openDB">Open DB object</button>
	<button id="addData">Add data</button>
	<button id="showData">Show Data</button>
	<button id="deleteData">Delete random row</button>
<div id="result">Results will show here.</div>
</html>
```

4. Add a `script` element to hold the code that interacts with the database.

```
<html>
	<button id="openDB">Open DB object</button>
	<button id="addData">Add data</button>
	<button id="showData">Show Data</button>
	<button id="deleteData">Delete random row</button>
<div id="result">Results will show here.</div>
	<script>
	</script>
</html>
```

Save your file with an .html extension. Now, you can open the HTML file in a web browser and see the interface shown in the introduction.

### Create the user interface: Basic interactivity
Let’s enable the buttons to respond to the code that we write. In this topic, complete the following steps by adding code inside the `script` element that you created in the **Create User Interface: Scaffolding** topic.

1. Create empty functions that should do some work when a user clicks on a button. Because they’re empty, the functions are not useful yet. You start to make them useful by adding code in the following topics. Create a function for each of the four buttons.

```
    ...
	<script>
		function openDB() {
		}
		function addData() {
		}
		function showData() {
		}
		function deleteData() {
		}
	</script>
```

2. Create a handle for the `div` element. Add the keyword `const` before the variable name so that the handle is read-only in the code.

```
    ...
	<script>
		const result = document.getElementById("result");

		function openDB() {
		}
		function addData() {
		}
		function showData() {
		}
		function deleteData() {
		}

	</script>
```

3. Create handles for each of the `button` elements. 
```
    ...
	<script>
		…

		function deleteData() {
		}

		open_button = document.getElementById("openDB");
		add_button = document.getElementById("addData");
		show_button = document.getElementById("showData");
		delete_button = document.getElementById("deleteData");
	</script>
```

4. Add an event listener to each of the `button` elements by using the handles that you created. Call the `addEventListener` method; listen for the `click` event, and invoke the corresponding function that you created in step 1.

```
    ...
	<script>
		...

		function deleteData() {
		}

		open_button = document.getElementById("openDB");
		add_button = document.getElementById("addData");
		show_button = document.getElementById("showData");
		delete_button = document.getElementById("deleteData");

		open_button.addEventListener("click", () => openDB());
		add_button.addEventListener("click", () => addData());
		show_button.addEventListener("click", () => showData());
		delete_button.addEventListener("click", () => deleteData());
	</script>
```

You are ready to start adding behaviour to the page. You can test the buttons by adding the following statement to any function that you called in step 1: `result.textContent = “It worked”;`.

Be sure to remove any test statements before you continue to the next topic.

### Open a database
Recall that IndexedDB is an asynchronous process, and requires you to create callback functions for each interaction. In this topic, you add code to the openDB() function that you created in **Create user interface: Basic interactivity** topic. You also create additional functions to handle callback operations from IndexedDB.

1. Inside your `script` element, create a global variable to hold a reference to your database.

```
	<script>
		let db;

		const result = document.getElementById("result");

		function openDB() {
		}
		...
	</script>
```

2. In your `openDB()` function, call the `open()` method on IndexedDB to open a database. Specify the name and the version number of the database schema. If the database does not exist, IndexedDB creates a database with the name that you specify in the request.

```
		let db;

		const result = document.getElementById("result");

		function openDB() {
		  req = indexedDB.open(“SimpleDB”, 1);
		}
```

3. Create empty functions to handle callback operations for the following conditions: success, error, and schema definition required.

```
		let db;

		const result = document.getElementById("result");

		function openDB() {
		  req = indexedDB.open(“SimpleDB”, 1);
		}

        function dbOkay() {
        }
        function err_handler() {
        }		
        function defineDBSchema() {
        }		
```

4. On your request object, specify the function to invoke for each condition. When IndexedDB invokes a callback function, it passes an `event` object that contains the result of the request. Pass the `event` object to the function.

```
        let db;
		
		const result = document.getElementById("result");

		function openDB() {
		  const req = indexedDB.open("SimpleDB", 1);
		  req.onsuccess = (event) => dbOkay(event);
		  req.onerror = (event) => err_handler(event);
		  req.onupgradeneeded = (event) => defineDBSchema(event);
		}

        function dbOkay() {
        }
        function err_handler() {
        }		
        function defineDBSchema() {
        }
```

5. Update your callback function signatures to accept the `event` object.

```
        ...
        function dbOkay(event) {
        }
        function err_handler(event) {
        }		
        function defineDBSchema(event) {
        }
```


6. If the open request is successful, assign the database reference to the global variable that you created in step 1. 

```
        ...
		function dbOkay(event) {
		  db = event.target.result;
        }
```

7. Print the status in the result `div` element that you created earlier. 
```
        ...
		function dbOkay(event) {
		  db = event.target.result;
		  result.textContent = “Got the DB”;
        }
```

8. Update your `err_handler()` function to report the error to the user.
```
        ...
		function err_handler(event) {
		  result.textContent = “The following error occurred: ” + event.target.errorCode;
        }
```

9. Before we define our database schema, let’s add another global constant to track the name of our Object Store object. This constant reduces the opportunity for bugs arising from mistyping the name of the ObjectStore. 

Add the following line between the `db` global variable and the `result` handle at the top of the `script` element.

```
	<script>
		let db;

		const STORE_NAME = “SimpleStore”;
		const result = document.getElementById(“result”);
		...
```

10. Specify your database schema in the `defineDBSchema()` function. Our database has a single table (SimpleTable), and no indexes. The records should have a `key` field named “id”, and the table should be responsible for incrementing the value of the field.

```
		function defineDBSchema(event) {
		  event.target.result.createObjectStore(STORE_NAME, {keyPath: “id”, autoincrement: true});
		}
```

Records in SimpleTable now have a key and any other value that you add when you commit a JSON object to that table. For more information about keys, see the IndexedDB Key Characteristics guide on MDN.

For a more sophisticated application, you should add a callback for a `versionchange` event and register it with `req.onversionchange`. This can be useful if the user has your application open in more than one tab, and loads a newer version of the database in one tab. The `versionchange` event gives you the opportunity to update your database schema before you try to write to the database while using an outdated schema. 

For more information about the `versionchange` event, see the API documentation on MDN.

11. Add a helper function to create a Transaction object and return a reference to the `SimpleTable` Object Store.

```
		...
		function defineDBSchema(event) {
		  event.target.result.createObjectStore(STORE_NAME, {keyPath: “id”, autoincrement: true});
		}

		function getStore() {
		  tx = db.Transaction(STORE_NAME, “readwrite”);
		  return tx.objectStore(STORE_NAME);
        }

	    ...
```

We can call the `getStore` function each time we want to perform an operation on the database. 

### Add data to the database
For the sake of simplicity, this task defines some static data to add to the database each time the user clicks on the **Add Data** button. This task adds code to the `getData()` function that you created earlier.

1. In the getData() function, call the `getStore()` function that you created in a previous task to retrieve a reference to the Object Store for SimpleTable.
```
		getData() {
		  var store = getStore();
        }
```

2. Create an array of data to add to SimpleTable. Under normal circumstances, this data would come from user input or some generated source. For brevity, let’s add static data.

```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
        }
```

3. Clear the content of the `div` element used to display results.
```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
        }
```


4. Let’s iterate over each of the rows in our array.
```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
			
		  });
        }
```

5. On each iteration, create a request to add the array element to the SimpleTable Object Store.
```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
		    req = store.add(row);
		  });
        }
```

6. Register a callback for the `onsuccess` event. In this case, we’ll create the callback function inline, instead of creating a separate function.
```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
		    req = store.add(row);
		    req.onsuccess = () => {

            };
		  });
        }
```

7. In the callback function, add a `p` element with the name of the row that you added to the database.

```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
		    req = store.add(row);
		    req.onsuccess = () => {
		      paragraph = document.createElement(“p”);
		      paragraph.textContent = “Added “ + row.name;

            };
		  });
        }
```

7. Append the `p` element to the results `div`.

```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
		    req = store.add(row);
		    req.onsuccess = () => {
		      paragraph = document.createElement(“p”);
		      paragraph.textContent = “Added “ + row.name;
                              results.append(paragraph);
            };
		  });
        }
```
7. Register the `err_handler()` function that you created earlier to respond to any `onerror` events.

```
		getData() {
		  var store = getStore();
		  var rows = [];
		  rows.push({name: “Apple”, weight: 102});
		  rows.push({name: “Banana”, weight: 125});
		  rows.push({name: “Orange”, weight: 141.7});
		  results.textContent = “”
		  rows.forEach((row) => {
		    req = store.add(row);
		    req.onsuccess = () => {
		      paragraph = document.createElement(“p”);
		      paragraph.textContent = “Added “ + row.name;
              results.append(paragraph);
            };
		    req.onerror = err_handler(event);
		  });
        }
```


### Retrieve data
This task creates the code to displays the data in a `table` element. 

You need two steps to retrieve data from the database: use a `cursor` object to move through an index of keys, and use each key to retrieve the corresponding record from the Object Store.

The `cursor` object sends an `success` event for each key it encounters. You can use the key to retrieve corresponding record from Object Store. 

In this task, you create a `populateTable()` function as a callback for the `cursor` object’s success event, and a `makeRow()` function as a callback for the Object Store’s `success` event.

<diagram>

1. Below the empty `addData()` function, add empty functions for the callbacks used in later steps in this task: `populateTable()`, `makeRow()`. You also need a `makeHead()` function to define the table header.

```
		...
		function showData() {
		}

		function makeHead() {
		}

		function populateTable() {
		}

		function makeRow() {
		}

```

2. In the `showData()` function, call the `getStore()` function that you created in a previous task to retrieve a reference to the Object Store for SimpleTable.

```
		function showData() {
		  var store = getStore();
		}
```

3. Create HTML elements for the table.

```
		function showData() {
		  var store = getStore();
		  var data_table = document.createElement("table");
		  var row_element = document.createElement("tr");
		  var td_element = document.createElement("td");
		}
```

4. Pass the `row_element` handle and the `td_element` handle to the `makeHead()` function to create a table header.

```
		function showData() {
		  var store = getStore();
		  var data_table = document.createElement("table");
		  var row_element = document.createElement("tr");
		  var td_element = document.createElement("td");

		  var head_row = makeHead(row_element, td_element);
		}
```

5. In the `makeHead()` function, update the parameter list to accept the row and cell elements passed from `showData()`, and define the header row and return the row.

```
		function makeHead(row, cell) {
		  var col_heads = ["ID", "Fruit", "Weight"];
		  var head_row = row.cloneNode(true);
		  col_heads.forEach((col) => {
		    column = cell.cloneNode(true);
		    column.textContent = col;
		    head_row.append(column);
		  });
		  return head_row;
		}
```

6. In the `showData()` function, add the table header to the `table` element.

```
		function showData() {
		  var store = getStore();
		  var data_table = document.createElement("table");
		  var row_element = document.createElement("tr");
		  var td_element = document.createElement("td");

		  var head_row = makeHead(row_element, td_element);
		  data_table.append(head_row);
		}
```

7. Create a request for a new `cursor` object from the Object Store. Register the `populateTable()` function as the callback for the `success` event. The `populateTable()` function is responsible for creating the table, so pass the relevant data: the cursor, the Object Store, and the HTML elements.

```
		function showData() {

		  ...

		  var req = store.openCursor();
		  req.onsuccess = (event) => populateTable(event, store, data_table, row_element, td_element);
```

8. In the `populateTable()` function, update the parameter list to accept the data passed from `showData()`

```
		function populateTable(cursor, store, dt, row, cell) {
		}
```


9. Retrieve the cursor. If it’s valid, get the associated record from the Object Store.
```
		function populateTable(cursor, store, dt, row, cell) {
		  var curs = cursor.target.result;
		  if (curs) {
		    req = store.get(curs.key);
		  }
		}
```

9. If the Object Store sends a `success` event, pass the relevant data to the `makeRow()` function: the key, the record data, the data table, and HTML row and cell elements. 
```
		function populateTable(cursor, store, dt, row, cell) {
		  var curs = cursor.target.result;
		  if (curs) {
		    req = store.get(curs.key);
		    req.onsuccess = (event) => makeRow(curs.key, event, dt, row, cell);
		  }
		}
```


10. In the `makeRow()` function, update the parameter list to accept the data passed from `populateTable()`. 

```
		function makeRow(key, db_values, dt, row, cell) {
		}
```

11. Retrieve the values from the record and create a new row. Append the row to the HTML table element.
```
		function makeRow(key, db_values, dt, row, cell) {
          dbv = db_values.target.result
          data_row = row.cloneNode(true);
          vals = [key, dbv.name, dbv.weight];
          vals.forEach((val) => {
		    column = cell.cloneNode(true);
		    column.textContent = val;
		    data_row.append(column);
          });
          dt.append(data_row);
		}
```

12. In the `populateTable()` function, ask for the next key from the cursor.
```
		function populateTable(cursor, store, dt, row, cell) {
		  var curs = cursor.target.result;
		  if (curs) {
		    req = store.get(curs.key);
		    req.onsuccess = (event) => makeRow(curs.key, event, dt, row, cell);
		    curs.continue();
		  }
		}
```


13. In the `showData()` function, Register the `err_handler()` function that you created in a previous task as the callback for the `error` event.

```
		function showData() {

		  ...

		  var req = store.openCursor();
		  req.onsuccess = (event) => populateTable(event, store, data_table, row_element, td_element);

		  req.onerror = (event) => err_handler(event);

```

### Delete data
You can delete any record for which you have a valid key. In this task, you choose a key at random, and delete the associated record.

1. In the `deleteData()` function, call the `getStore()` function that you created in a previous task to retrieve a reference to the Object Store for SimpleTable.

```
		function deleteData() {
			var store = getStore();
		}
```

2. Create a variable to store the number of records in the Object Store.
```
		function deleteData() {
			var store = getStore();
			var max_records = 0;
		}
```

3. Create a request to count the number of records in the Object Store.

```
		function deleteData() {
			var store = getStore();
			var max_records = 0;
			var req = store.count();
		}
```

4. For the sake of brevity, create an inline function as a callback for the `success` event.
```
		function deleteData() {
			var store = getStore();
			var max_records = 0;
			var req = store.count();
			req.onsuccess = (event) => {
			};
		}
```

5. If the number of records is greater than zero, choose one at random, and create a request to delete it.

```
		function deleteData() {
			var store = getStore();
			var max_records = 0;
			var req = store.count();
			req.onsuccess = (event) => {
				max_records = event.target.result;
				if (max_records > 0) {
					del_record = Math.floor(Math.random()*max_records);
					del_req = store.delete(del_record);
				}
			};
		}
```

6. If the request is successful, regenerate the data table.

```
		function deleteData() {
			var store = getStore();
			var max_records = 0;
			var req = store.count();
			req.onsuccess = (event) => {
				max_records = event.target.result;
				if (max_records > 0) {
					del_record = Math.floor(Math.random()*max_records);
					del_req = store.delete(del_record);
					del_req.onsuccess = (event) => {
						showData();
					};
				}
			};
		}
```

## Next steps
You now have a fully functional web page that stores and retrieves data by using the brower’s IndexedDB API. You can play with this example, try to generate error conditions, and extend it as needed.

MDN maintains a list of libraries that wrap the IndexedDB API for a variety of use-cases. If your application uses the Promise pattern of asynchronous programming (async/await), Jake Archibald’s IDB library is a common tool.



