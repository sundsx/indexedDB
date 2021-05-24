# indexedDB
Homework 12 For OOP Programming paradigm

# intro
Object-oriented programming (OOP) is a programming paradigm that allows you to define software objects
capable of interacting with each other through the exchange of messages.
Particularly suitable in contexts in which interdependence relationships can be defined between the concepts to be modeled
(containment, use, specialization), an area that more than others manages to exploit the advantages of object-oriented programming is
that of graphical interfaces.

Another context is in NOSQL pattern design. Or where you need dynamic (not fixed) tables.

For a demonstration, I describe a simple app that manages Music album. 
We will create a simple personal vinyl database.
The user will insert all the vinyls he owns into his database.

The project is intended to be domistrative and therefore not implemented of all possibilities. However, it is free to be modified for your own purpose 

The operations to be performed, CRUD, are to insert, read, update and delete the data.
For this example I will use the indexedDB database, a simple NOSQL db in browser

IndexedDB is a transactional database system, like an SQL-based RDBMS. However, unlike SQL-based RDBMSes, which use fixed-column tables,
IndexedDB is a JavaScript-based object-oriented database.
IndexedDB lets you store and retrieve objects that are indexed with a key; any objects supported by the structured clone algorithm can be stored.
You need to specify the database schema, open a connection to your database, and then retrieve and update data within a series of transactions.
CRUD operations.


The structure of db are 5 simple fields: id, title, band, genre, release.

# First we need to create an object to store

		var db;
		window.onload = function() {

		// In the following line, you should include the prefixes of implementations you want to test.
		window.indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB;

		// Moreover, you may need references to some window.IDB* objects:
		window.IDBTransaction = window.IDBTransaction || window.webkitIDBTransaction || window.msIDBTransaction;
		window.IDBKeyRange = window.IDBKeyRange || window.webkitIDBKeyRange || window.msIDBKeyRange;
		// (Mozilla has never prefixed these objects, so we don't need window.mozIDB*)

		// Let us open our database
		var DBOpenRequest = window.indexedDB.open("musicArchive", 4);

		DBOpenRequest.onupgradeneeded = function (e) {

		db = DBOpenRequest.result;

		var objectdb = db.createObjectStore("vinyl", {keyPath: 'id', autoIncrement : true });

		objectdb.createIndex('index_title','title', {unique : true});
		objectdb.createIndex('index_band','band', {unique : false});
		objectdb.createIndex('index_genre','genre', {unique : false});
		objectdb.createIndex('index_release','release', {unique : false});
		};

		DBOpenRequest.onsuccess = function(event) {
		note.innerHTML += '<li>Database initialised.</li>';

		// store the result of opening the database in the db variable.
		// This is used a lot below
		db = DBOpenRequest.result;

		// Run the displayData() function to load data from database
		displayData();
		};
		}
		
		
# Disply data on table

	function displayData() {

	// first clear the content of the task list so that you don't get a huge long list of duplicate stuff each time
	//the display is updated.
	let bodyTable = document.getElementById('songtable');
	while(bodyTable.hasChildNodes())
	{
	bodyTable.removeChild(bodyTable.firstChild);
	}

	let taskList = document.querySelector("#taskList");
	taskList.innerHTML = "";

	let table = document.querySelector("#songtable");

	// Open our object store and then get a cursor list of all the different data items in the IDB to iterate through
	let objectStore = db.transaction('vinyl', "readonly").objectStore('vinyl');
	objectStore.openCursor().onsuccess = function(event) {
	let cursor = event.target.result;
	// if there is still another cursor to go, keep runing this code
	if(cursor) {
	// create a list item to put each data item inside when displaying it
	const listItem = document.createElement('li');

	let tr = document.createElement("TR");

	const tdId = document.createElement("TD");
	const tdTitle = document.createElement("TD");
	const tdBand = document.createElement("TD");
	const tdGenre = document.createElement("TD");
	const tdRelease = document.createElement("TD");
	const tdDel = document.createElement("TD");
	const tdEdit = document.createElement("TD");

	tdId.innerHTML = cursor.value.id;
	tdTitle.innerHTML = cursor.value.title;
	tdBand.innerHTML = cursor.value.band;
	tdGenre.innerHTML = cursor.value.genre;
	tdRelease.innerHTML = cursor.value.release;

		// del button
	const delBtn = document.createElement('button');
	delBtn.setAttribute('class', 'btn btn-light btn-sm');

	tdDel.appendChild(delBtn);

	delBtn.innerHTML = '🧹';
	// here we are setting a data attribute on our delete button to say what task we want deleted if it is clicked!
	delBtn.setAttribute('data-task', cursor.value.id);

	delBtn.onclick = function(event) {
	//  deleteItem(event);
	let dataToTrash = event.target.getAttribute('data-task');
	deleteResult(event, dataToTrash);
	}

		// edit button

	const editBtn = document.createElement('button');
	editBtn.setAttribute('class', 'btn btn-light btn-sm');

	tdEdit.appendChild(editBtn);
	editBtn.innerHTML = '✏️';
	// here we are setting a data attribute on our delete button to say what task we want deleted if it is clicked!
	editBtn.setAttribute('data-task', cursor.value.id);

	editBtn.onclick = function(event) {
	 //  deleteItem(event);
	let dataToTrash = event.target.getAttribute('data-task');
	 //  updateResult(event, dataToTrash);
	preUpdate(event, dataToTrash);
	}

	tr.appendChild(tdId);
	tr.appendChild(tdTitle);
	tr.appendChild(tdBand);
	tr.appendChild(tdGenre);
	tr.appendChild(tdRelease);
	tr.appendChild(tdDel);
	tr.appendChild(tdEdit);

	table.append(tr);

	// continue on to the next item in the cursor
	cursor.continue();

	// if there are no more cursor items to iterate through, say so, and exit the function
	} else {
	note.innerHTML += '<li>Entries all displayed.</li>';
	}
	

# Delete operation

  		function deleteResult(event, dataTrash) {
      let transaction = db.transaction(["vinyl"], "readwrite");
      let objectStore = transaction.objectStore("vinyl");

      objectStore.openCursor().onsuccess = function(event) {

        let dataTask = dataTrash;

        const cursor = event.target.result;
        if(cursor) {
          if(cursor.value.id === parseInt(dataTask)) {
            const request = cursor.delete();
            request.onsuccess = function() {
              console.log('Deleted' + cursor.value.title);
              displayData();

            };
          }
					
          cursor.continue();
					
        } else {
				
          console.log('Entries displayed.');
					
        }
      };
    };
		
# Populate form with selected data to update

		function preUpdate(event, dataTrash){

      let id = dataTrash;

      const transaction = db.transaction(["vinyl"], "readonly");
      const objectStore = transaction.objectStore("vinyl");

      objectStore.openCursor().onsuccess = function(event) {

      let dataTask = dataTrash;

      const cursor = event.target.result;
      if(cursor) {
      if(cursor.value.id === parseInt(dataTask)) {

      document.querySelector('#editId').value = cursor.value.id;
      document.querySelector('#editTitle').value = cursor.value.title;
      document.querySelector('#editBand').value = cursor.value.band;
      document.querySelector('#editGenre').value = cursor.value.genre;
      document.querySelector('#editRelease').value = cursor.value.release;

      $('#modal').modal('show');

      }
      cursor.continue();

      } else {
        console.log('Entries displayed.');
      }

    };
		}
		
# Save modified data

		  function updateAlbum(event) {

    const transaction = db.transaction(["vinyl"], "readwrite");
    const objectStore = transaction.objectStore("vinyl");

    const editId = document.querySelector('#editId').value;
    const editTitle = document.querySelector('#editTitle').value;
    const editBand = document.querySelector('#editBand').value;
    const editGenre = document.querySelector('#editGenre').value;
    const editRelease = document.querySelector('#editRelease').value;

    objectStore.openCursor().onsuccess = function(event) {

      let dataTask = editId;

      const cursor = event.target.result;
      if(cursor) {
        if(cursor.value.id === parseInt(dataTask)) {

          const updateData = cursor.value;

          updateData.title = editTitle;
          updateData.band = editBand;
          updateData.genre = editGenre;
          updateData.release = editRelease;

          const request = cursor.update(updateData);
          request.onsuccess = function() {
            console.log('A better album year?');
          };
          document.getElementById("editForm").reset();
          $('#modal').modal('hide');

          displayData();

        }
        cursor.continue();
      } else {
        console.log('Entries displayed.');
      }
    };

  }
