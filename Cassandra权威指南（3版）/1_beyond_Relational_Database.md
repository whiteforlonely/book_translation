### Sharding And Shared-Nothing Architecture



*If you can't split it, you can't scale it.*
&emsp;&emsp;---Randy Shoup Distinguished Architect, eBay



Another way to attempt to scale a relational database is to introduce sharding to your architecture. This has been used to good effect at large websites such as eBay, witch support billions of SQL queries a day, and in other modern web applications. The idea here is that you split the data so that instead of hosting all of it on a single server or replicating all of the data on all of the servers in a cluster, you divide up portions of the data horizontally and host them each separately.

For example, consider a large customer table in a relational database, The least disruptive thing (for the programming staff, anyway) is to vertically scale by adding CPU, add memory, and getting faster hard drivers, but if you continue to be successful and add more customers, at some point (perhaps into the tens of millions of rows), you'll likely have to start thinking about how you can add more machines, When you do so, do  you  jsut copy the data so that all of the machines have it? Or do you instead divide up that single customer table so that each database has only some f the records, with their order preserved? Then when clients execute queries, They put load only on the machine that has the record they're looking for, with no load on the other machines.

It seems clear that in order to shard, you need to find a good key by which to order you records. for example, you could divide your customer records across 29 machines, one for each letter of the alphabet, with each hosting only the records for customers whose last names start with that paricular letter, It's likely this is not a good strategy, however--there probably aren't many last names that begin with "Q" or "Z", so those machines will sit idle while the "J", "M", and "S" machine spike, you could shard according to something numeric, like phone number, "member since" date, or the name of the customer's state, It all depends on how your specific data is likely to be distributed.

There are three basic strategies for determining shard structure:

##### *Feature-based shard or functional segmentation*

This is the approach taken by Randy Shoup, Distinguished Architect at eBay, who in 2006 helped bring the site's architecture into maturity to support many billions of queries per day. Using this strategy, the data is split not by dividing records in a single table (as in the customer example discussed earlier), but rather by splitting into separate databases the features that don't overlap with each other very much, For example, at eBay, the users are in one shard, and the items for sale are in another, Theis approach depends on understanding your domain so that you can segment data cleanly.

##### *Key-based sharding*
In this approach, you find a key in your data that will evenlydistribute it across shards, So instead of simply storing one letter of the alphabet for each server as in the (naive and improper) earlier example, you use a one-day hash on a key data element and distribute data across machines according to the hash. It is common in this strategy tofid time-based or numeric keys to hash on.

##### *Lookup table*
In this approach also known as *directory-based sharding*, one of the nodes in the cluster acts as a "Yellow Pages" directory and looks up which node has the data you're trying to access. This has two obvious disadvantages. The first is that you'll take a performance hit every time you have to go through  the loopup table as an additional hop. The second is that the lookup table not only becomes a bottleneck, but a single point of failure.

Sharding can minimize contention depending on your stargegy and allows you not just to scale horiozontally, but then to scale precisely, as you  can add power to the particular shards that need it.

Sharding could be termed a kind of *shared-nothing* architecture that's specific to databases, A shared-nothing architecture is one in which there is no centralized (shared) state, but each node is a distributed system is independent, so there is no client contention for shared resources.

Shared-nothing architecture was more recently popularized by Google, with has written systems such as its Bigtable database and its Map Reduce implementation that do not share state, and are therefore capable of near-infinite scaling. The Cassandra database is a shared-nothing architecture, as it has no central controller and no notion of primary/secondary replicas; all of its nodes are the same.

> ##### More on Shared-Nothing Architecture
> 
> The term was first coined by Michael Stonebraker at the University of California at Berkeley in his 1986 paper "The Case for Shared Nothing". It's only a few ages. If you take a look, you'll see that many of the features of shared-nothing distributed data architecture, such as ease of high availability and the ability to scale to a very large number of machines, are the very things that Cassandra excels at.

Many nonrelational databases offer this automatically and out of the box is very handy; creating adn maintaining custom data shards by hand is a wicked proposition. For example, MongoDB, which we'll discuss later, provides auto-sharding capabilities to manager failover and node balancing, It's good to understand sharding in terms of data architecture in general, but especially in terms of Cassandra more specifically. Cassnadra uses an approach similar to key-based sharding to distribute data across nodes, but does so automatically.

###  Web Scale

In summary, relational databases are very good at solving certain data storage problems, but because of their focus, they also can create problems of their own when it’s time to scale. Then, you often need to find a way to get rid of your joins, which means denormalizing the data, which means maintaining multiple copies of data and seriously disrupting your design, both in the database and in your application. Further, you almost certainly need to find a way around distributed transactions, which will quickly become a bottleneck. These compensatory actions are not directly supported
in any but the most expensive RDBMSs. And even if you can write such a huge check, you still need to carefully choose partitioning keys to the point where you can never entirely ignore the limitation.

Perhaps more importantly, as you see some of the limitations of RDBMSs and conse quently some of the strategies that architects have used to mitigate their scaling issues, a picture slowly starts to emerge. It’s a picture that makes some NoSQL solutions seem perhaps less radical and less scary than you may have thought at first, and more like a natural expression and encapsulation of some of the work that was already being done to manage very large databases.

Because some of the inherent design decisions in RDBMSs, it's not always as easy to scale as some other, more recent possibilities that take the structure of web into consideration. However, it's not only not the structure of the web you need to consider, but also its phenomenal growth, because more and more data becomes available, you need architectures that allow your organization to take advantage of this data in near real time to support decision making, and to offer new and more powerful features and capabilities to your customers.

> ##### Data Scale, Then and Now
> It has been said, though it is hard to verify, that the 17th-century English poet John Milton had actually read every published book on the face of the earth. Milton knew many languages (he was even learning Navajo at the time of his death), and given that the total number of published books at that time was in the thousands, this would have been possible. The size of the world’s data stores have grown somewhat since then.

With the rapid growth in the web, there is great variety to the kinds of data that need to be stored, processed, and queried, and some variety to the businesses that use such data. Consider not only customer data at familiar retailers or suppliers, and not only digital video content, but also the required move to digital television and the explosive growth of email, messaging, mobile phones, RFID, Voice Over IP (VoIP) usage, and the Internet of Things (IoT). Companies that provide content-and the third-party value-add businesses built around them-requir very scalable data solutions. Consider too that a typical business application developer or database administrator may be used to thinking of relational databases as the center of the universe. You might then be surprised to learn that within corporations, around 80% of data is unstructured.

### The Rise of NoSQL

The recent interest in nonrelational databases reflects the growing sense of need in the software development community for web scale data solutions.  The term "NoSQL" began gaining popularity around 2009 as a shorthand way of describing these databases. The term has historically been the suject of much debate, but a consensus has emerged that the term refers to nonrelational databases that support "not only SQL" semantics.

Various experts have attempted to organize these databases in a few broad categories; let's examine a few of the most common:

*Key-value stores*
In a key-value store, the data items are keys that have a set of attributes. All data relevant to a key is stored with the key; data is frequently duplicated. Popular key-value stores include Amazon's Dynamo DB, Riak, and Voldemort. Additionally, many popular caching technologies act as key-value stores, including Oracle Coherence, Redis, and Memcached.

*Column stores*
In a column store, also known as a wide-column  store or colum-oriented store, data is stored by column rather than by row. For example, in a column sotre, all customer

*Document stores*



