---
title: "From LocalStorage to indexedDB"
date: "2021-09-07"
categories: 
  - "tech"
tags: 
  - "github"
  - "indexeddb"
  - "localstorage"
image: "53b5f-screen-shot-2021-09-10-at-4.34.50-pm.png"
---

While rewriting an offline capable webapp, I decided to update the offline storage mechanism from the very easy to use LocalStorage, to [indexedDB](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API), a real client database.

When writing the app, initially I wasn't concerned with building the most scalable app, as much as getting something working. Therefore, I went with the easiest to use and fastest to implement technology, LocalStorage. It completely met my needs at the time and I was very happy with it. But, as the app was finished and I began storing more data, I ran into two shortcomings. First, the possibility of going over the 5mb of text data limitation, and second the concern of JS thread blocking when reading large files. Both are good problems to have, meaning my app was working and being used more, however it highlighted that I was outgrowing the capacity of LocalStorage.

If you have smaller storage requirements (<5mb), and simply need a persistent key-value store, I would still recommend using LocalStorage. At small sizes, you will not notice either of the aforementioned issues.

So, the primary benefits to using a datastore like indexedDB are: larger data storage limits (50MB), non-blocking operations, and the ability to do db operations beyond simple read/writes. In my case, the first two alone are enough to switch over.

You can find detailed usage [instructions](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API/Using_IndexedDB) on Mozillas' page. At a high level the steps are:

Check that the browser has IndexedDB support:

```
if (window.indexedDB) { ... }
```

Request to open a named database:

```
const request = window.indexDB.open([database name], [version])
```

this returns an IDBOpenDBRequest object which you will define actions for lifecycle events on:

```
.onerror(e) { ... } //failed to open
.onsuccess(e)  { ... } // set your database, ie db = event.target.result
.onupgradeneeded(e)  { ... } //create or update your db.  Set your db as above, and create an object store with the collection name, and primary key.  Here we can also call .createIndex on our newly created object store to set other indexable fields. 
```

There are basically three levels. A named database itself, an objectStore which is like a collection, and a transaction which is basically the action you will invoke on the objectStore. To add or delete data you'll follow a pattern of first using your objectStore and calling its transaction method to call add with the record you want to insert, or delete with the id of the record you want to delete. Since it's an async process, you'll also need to handle oncomplete and onerror events that will be triggered on the transaction object.

To read from the database, you'll follow the above pattern to use your database to create a transaction against a list of ObjectStores and specify the type of access, then use that and specify a single ObjectStore. Yes, it's a little weird and feels like an unnecessary step to create the transaction against a list, all with the same type of acccess (ie readwrite), then to pull a single ObjectStore out. I suppose it's optimized for the case that you'll be working with multiple collections at the same time, but odd that you'd set them all for the same access type.

Anyway, with that ObjectStore, you'll define handlers for its' onerror and onsuccess events. In the handlers, you'll use the event.target.result to get the cursor. Once finished with a record, you'll call cursor.continue() for the next record.

For database updates, you'll follow the pattern of getting an record, updating it's value(s), and then calling objectStore.put(newRecord) to send the update.

Here is an example webpage of a contact form using indexedDB which show how to instantiate a db instance, read from the database, write, and delete. It's very basic and is only to show functionality of indexedDB.

## index.html

```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link rel="stylesheet" href="./styles.css" />
    <title>Example using indexedDB</title>
  </head>
  <body>
    <h1>Contact Manager</h1>
    <form action="" class="new-contact">
      <div>
        <label for="" required>Enter your name:</label>
        <input type="text" id="name" required />
      </div>
      <div>
        <label for="" required>Enter your number:</label>
        <input type="text" id="number" required />
      </div>
      <button id="addName">Save Contact</button>
    </form>
    <section class="contacts">
      <ul></ul>
    </section>
    http://index.js
  </body>
</html>
```

## index.js

```
let db;
const nameInput = document.querySelector('#name');
const numberInput = document.querySelector('#number');
const form = document.querySelector('form');
const list = document.querySelector('ul');

window.onload = () => {
  const request = window.indexedDB.open('contacts', 1);

  request.onerror = (e) => {
    console.log('Database failed to open');
  };

  request.onsuccess = (e) => {
    console.log('got our db');
    db = e.target.result;
    displayData();
  };
  request.onupgradeneeded = (e) => {
    db = e.target.result;
    const objectStore = db.createObjectStore('contacts', { keyPath: 'id', autoIncrement: true });
    objectStore.createIndex('name', 'name', { unique: false });
    objectStore.createIndex('number', 'number', { unique: false });

    console.log('setup complete');
  };

  form.onsubmit = addData;

  function addData(e) {
    e.preventDefault();
    const newItem = { name: nameInput.value, number: numberInput.value };
    const transaction = db.transaction(['contacts'], 'readwrite');
    const objectStore = transaction.objectStore('contacts');
    const request = objectStore.add(newItem);
    request.onsuccess = () => {
      nameInput.value = '';
      numberInput.value = '';
    };
    transaction.oncomplete = () => {
      console.log('transaction completed on the db');
      displayData();
    };
    transaction.onerror = () => {
      console.log('transaction failed on the db');
    };
  }

  function deleteItem(e) {
    e.preventDefault();
    const transaction = db.transaction(['contacts'], 'readwrite');
    const objectStore = transaction.objectStore('contacts');
    const request = objectStore.delete(Number(e.target.parentNode.getAttribute('data-contact-id')));
    transaction.oncomplete = () => {
      e.target.parentNode.parentNode.removeChild(e.target.parentNode);
      console.log('deleted record from the db');
      if (!list.firstChild) {
        const listItem = document.createElement('li');
        listItem.textContent = 'no contacts store';
        list.appendChild(listItem);
      }
    };
    transaction.onerror = () => {
      console.log('transaction failed on the db');
    };
  }

  function displayData() {
    while (list.firstChild) {
      list.removeChild(list.firstChild);
    }

    const objectStore = db.transaction('contacts').objectStore('contacts');
    objectStore.openCursor().onsuccess = (e) => {
      const cursor = e.target.result;
      if (cursor) {
        const listItem = document.createElement('li');
        const name = document.createElement('p');
        const number = document.createElement('p');

        listItem.appendChild(name);
        listItem.appendChild(number);
        list.appendChild(listItem);

        name.textContent = cursor.value.name;
        number.textContent = cursor.value.number;

        listItem.setAttribute('data-contact-id', cursor.value.id);

        const deleteButton = document.createElement('button');
        deleteButton.textContent = 'Delete';
        deleteButton.onclick = deleteItem;
        listItem.appendChild(deleteButton);

        cursor.continue();
      } else if (!list.firstChild) {
        const listItem = document.createElement('li');
        listItem.textContent = 'No contacts stored';
        list.appendChild(listItem);
      }
      console.log('contacts displayed');
    };
  }
};
```

You can find the full example on [github](https://github.com/paultman/indexedDBExample).

You can also have a look at the official w3c api docs [here](https://w3c.github.io/IndexedDB/).

I also wrote a follow up [post](https://paultman.wordpress.com/best-library-for-indexeddb-localforage-idb-keyval-or-idb/) evaluating indexedDB wrapper libraries.
