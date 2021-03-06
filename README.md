Unique Record Data Store

Purpose:

1) Create a web service that accepts HTTP POSTs with variable-length strings as the request body. If this string has been sent in a request before, return a negative response. If the string has never been seen before, return a positive response and store the string.

2) Retrieve all strings posted during the life of the service.

Restrictions and Caveats:

- Runs on a single, dedicated machine
- No restrictions on OS
- Memory/storage will be unable to hold all records, so store as many possible until the drive is full

Requirements:

mongodb must be installed and mongod must be running

Usage (soon to change):

python uniqueds/mongo_store.py

Developer Notes:

When starting to develop this application, I began by trying to understand the full requirements, which are distilled above. After that, I examined different ways of approaching the problem. Working directly with the filesystem would minimize space overhead, so this was the approach I examined initially. I decided against this after realizing that, which this would maximize space, it would also require me to implement my own locking mechanism to ensure that the web service wouldn't run into thread issues. This also would be less scalable, should the service ever be expanded to more than one box. I decided that working with a database would be best, and decided to use MongoDB for two reasons. The first being that I was unsure of my schema structure in the beginning, plus I needed a database that would work easily with arbitrarily long strings (Mongo supports up to 4MB single records. This was large enough for my initial tests. More testing is required to see how this scales beyond that.) Mongo satisfied both these requirements with minimal configuration requirements. The second reason was that I had never used MongoDB before, and saw this as an excuse to test it out.

After deciding on my storage medium, the rest of the task was fairly simple. I am familiar with Python as a language, but I don't have the depth of experience with its libraries like I do in Java. So after some research, I decided to use the wsgiref library build into Python to handle the web service, pymongo for database interface, and hashlib to generate checksums. The acutal application is very simple. It checks whether the request is a POST or GET. 

POST requests are treated as insertions into the unique store. Using constructs found in MongoDB I was able to write the lookup/insertions as a single transaction, all handled by the database. To reduce the ammount of scans necessary, in addition to the original content, each record is also inserted with the content size and an MD5 checksum. While this does reduce the ammount of actual content that can be stored on the disk, it also allows us to index the database for faster lookup times and better request handling. Theoretically, if we wanted to make the tradeoff of more stored records with less throughput, we could remove the size and checksum, but I don't believe that the increase would be that large (profiling would provide more info on this).

GET requests return the full list of records in the database in JSON form.

------------
2013/10/28 - C.Hughes - After further reading, I'm concerned about how MongoDB does it's memory allocation and whether that will affect the total number of records able to be stored. I'm going to try to create a version of this using Postgres + File System and compare. This will probably be a tradeoff of memory vs throughput efficiency, because the Postgres version will require an explicit mutex lock to ensure transactionality when dealing with the filesystem.
