---
title: FileHandle API
slug: Web/API/File_Handle_API
tags:
  - API
  - Files
  - Reference
---
{{non-standard_header}}

The FileHandle API allows for the manipulating of files, including creating files and modifying their content (unlike the [File API](/en-US/docs/Web/API/File)). Because the files manipulated through that API can be physically stored on the device, the editing part uses a turn-based locking mechanism in order to avoid race issues.

## API Overview

This API is based on the following interfaces:

- {{domxref("IDBDatabase.createMutableFile")}} (was called {{domxref("IDBDatabase.mozCreateFileHandle")}}.)
- {{domxref("IDBMutableFile")}} (was previously {{domxref("FileHandle")}}.)
- {{domxref("LockedFile")}}
- {{domxref("FileRequest")}}

It also has connections with the File API, especially the {{domxref("File")}} and {{domxref("Blob")}} interfaces.

## Basic operations

### Create a FileHandle

Because the intent is to allow the storage of files through IndexedDB, creating a {{domxref("IDBMutableFile")}} instance requires an [IndexedDB Database](/en-US/docs/Web/API/IDBFactory#open).

```js
var IDBReq = indexedDB.open("myFileStorageDataBase");

IDBReq.onsuccess = function(){
  var DB = this.result;
  var buildHandle = DB.createMutableFile("test.txt", "plain/text");

  buildHandle.onsuccess = function(){
    var myFileHandle = this.result;
    console.log('handle', myFileHandle);
  };
};
```

{{domxref("IDBDatabase.createMutableFile","createMutableFile()")}} takes two arguments: a name and an optional type. Both of these are just descriptive and are not used by the database. However, they are important for the {{domxref("FileHandle")}} object as it can generate {{domxref("File")}} objects which inherit their own {{domxref("File.name","name")}} and {{domxref("File.type","type")}} from those values. That said, as the name does not match any real filename it can be an empty string, for example, and it doesn't even have to be unique.

> **Note:** the above code only creates a "temporary file" that exists only while you hold the {{domxref("FileHandle")}} instance. If you want a file to survive a page refresh/app relaunch, you need to store the handle in a more permanent location, like the database itself. See [File storage](#file_storage) below to learn more about this.

### Perform read and write operations

To read or write within a handled file, it is required to get a {{domxref("LockedFile")}}. The {{domxref("FileHandle.open()")}} method provides such an object which can be `readonly` or `readwrite`. Any attempt to perform a write action on a `readonly` {{domxref("LockedFile")}} object will fail.

#### Writing

There are three possible writing operations on a locked file:

- {{domxref("LockedFile.write","write")}} : It's an arbitrary writing method which starts writing in the file at the {{domxref("LockedFile.location")}} byte.
- {{domxref("LockedFile.append","append")}} : This operation always writes content at the end of the file.
- {{domxref("LockedFile.truncate","truncate")}} : This operation keeps the nth-first bytes of the file and removes the rest.

```js
// Get a LockedFile object from the handle
var myFile = myFileHandle.open('readwrite');

// Start a writing operation
var writing = myFile.append('Some content');

writing.onsuccess = function () {
  console.log('Writing operation successful');
}

writing.onerror = function () {
  console.log('Something goes wrong in the writing process: ' + this.error);
}
```

#### Reading

It's possible to directly write the content of a {{domxref("LockedFile")}} object without using an intermediate {{domxref("File")}} object and a {{domxref("FileReader")}} object. The {{domxref("LockedFile")}} interface provides a {{domxref("LockedFile.readAsText","readAsText")}} method and a {{domxref("LockedFile.readAsArrayBuffer","readAsArrayBuffer")}} method.

Those two methods expect a size to indicate how many bytes must be read starting at the {{domxref("LockedFile.location")}} byte.

To read the whole file, it is required to know its size. This information (as well as the date of its last modification) can be retrieved through the {{domxref("LockedFile.getMetadata()")}} method.

```js
// Get a LockedFile object from the handle
var myFile = myFileHandle.open('readwrite');

// Retrieve the size of the file
var getmeta = myFile.getMetadata();

getmeta.onsuccess = function () {
  var size = this.result.size;

  // The reading operation will start with the byte at index 0 in the file
  myFile.location = 0;

  // Start a reading operation for the whole file content
  var reading = myFile.readAsText(size);

  reading.onsuccess = function () {
    console.log('The content of the file is:');
    console.log(this.result);
  }

  reading.onerror = function () {
    console.log('Something goes wrong in the reading process: ' + this.error);
  }
}
```

### File snapshot

In many cases it can be handy to get a snapshot of the file. For example, there are many APIs that expect {{domxref("Blob")}} or {{domxref("File")}} objects such as {{domxref("FileReader")}} (which can be easier to use to read the whole file) or {{domxref("XMLHttpRequest")}}.

It's possible to get a {{domxref("File")}} object representing the current state of the file handled by the {{domxref("FileHandle")}} object by using the {{domxref("FileHandle.getFile","getFile")}} method. Such a {{domxref("File")}} object is completely desynchronized from the original file, which means any change made to that object will never be reflected to the handled file as well as any change made to the handled file will never be pushed to the {{domxref("File")}} object.

```js
var mySnapshot = null;
var request = myFileHandle.getFile();

request.onsuccess = function () {
  mySnapshot = this.result;
}
```

### Managing progress

All the methods from the {{domxref("LockedFile")}} interface return a {{domxref("FileRequest")}} object. Such an object is basically a {{domxref("DOMRequest")}} with an extra power: it allows to monitor the progress of an operation. Sometimes writing and reading operations can be very long, therefore it is a good idea to monitor the operation to provide feedback to the user. Such monitoring can be done using the {{domxref("FileRequest.onprogress")}} event handler.

```js
var progress = document.querySelector('progress');
var myFile   = myFileHandle.open('readonly');

// Let's read a 1GB file
var action   = myFile.readAsArrayBuffer(1000000000);

action.onprogress = function (event) {
  if (progress) {
    progress.value = event.loaded;
    progress.max   = event.total;
  }
}

action.onsuccess = function () {
  console.log('Yeah \o/ Just read a 1GB file');
}

action.onerror = function () {
  console.log('Oops :( Unable to read a 1GB file')
}
```

## File storage

When a file handle is created, the associated file only exists as a "temporary file" as long as you hold the {{domxref("FileHandle")}} instance. If you want a file to survive a page refresh/app relaunch, you need to store the handle in a database (not necessarily the one used to create the {{domxref("FileHandle")}} object).

```js
var IDBReq = window.indexedDB.open('myFileStorageDataBase');

// If necessary, let's create a datastore for the files
IDBReq.onupgradeneeded = function () {
  this.result.createObjectStore('files');
}

IDBReq.onsuccess = function () {
  var DB = this.result;

  // Let's create a new file
  var handleReq = DB.mozCreateFileHandle("test.txt", "plain/text");

  handleReq.onsuccess = function () {
    var myFileHandle = this.result;
    var store = DB.transaction(['files'], 'readwrite').objectStore('files');

    // Let's store the file permanently
    // HINT: it could be handy to use the file name as the storage key
    var storeReq = store.add(myFileHandle, myFileHandle.name);

    storeReq.onsuccess = function () {
      console.log('The file has been successfully stored and can be retrieved anytime.')
    }
  }
}
```

A file stored that way is physically put on the device. The database itself only stores a pointer to that file. It means that if the {{domxref("FileHandle")}} object is stored several times in several DBs or several data stores, all those objects will reference the same unique file. This is not a problem because to access the file, a {{domxref("LockedFile")}} object is required and operations on such object are performed in [isolation](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29), meaning that once a {{domxref("LockedFile")}} is active, all operations of this {{domxref("LockedFile")}} are guaranteed to happen sequentially on the underlying file without being interleaved with operations from other {{domxref("LockedFile")}}.

### Secured write operation

For performance reasons, write (and read) operations are done in memory. Periodically, the results of those operation are asynchronously flushed to the device storage area. If for some reason a problem occurs before that, you can lose the results of some operations. To avoid that problem, you can force the data to be flushed by using the {{domxref("LockedFile.flush()")}} method. Once this method has been successfully called, you can be sure your change on the file will be safe.

```js
// Get a LockedFile object from the handle
var myFile = myFileHandle.open('readwrite');

// Start a writing operation
var writing = myFile.append('Some content');

writing.onsuccess = function () {
  console.log('Writing operation successful');

  var saving = myFile.flush();

  saving.onsuccess = function () {
    console.log('The file has been successfully stored');
  }
}

writing.onerror = function () {
  console.log('Something goes wrong in the writing process: ' + this.error);
}
```

## API Compatibility

### Why a different API than FileWriter?

The [FileWriter specification](https://dev.w3.org/2009/dap/file-system/file-writer.html) defines FileWriters, objects aiming at representing editable files. [Discussions on public-webapps](https://lists.w3.org/Archives/Public/public-webapps/2012JanMar/0886.html) led to the conclusion that the API would behave poorly in the case of different entities writing concurrently to the same file. The outcome of this discussion is the FileHandle API with its LockedFile and transaction mechanism.

## Specifications

A formal specification draft is being written. As it does not fully match the current implementation, be warned that the implementation and/or the specification will be subject to changes.

| Specification                        | Status                           | Comment        |
| ------------------------------------ | -------------------------------- | -------------- |
| {{SpecName('FileSystem')}} | {{Spec2('FileSystem')}} | Draft proposal |

## Browser compatibility

Supported in Firefox 15.
