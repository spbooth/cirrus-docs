The Object Store
================

In addition to the lustre file-system Cirrus also has access to an Object-Store system. 
This web-service provides an additional place for you to store your data but it works in a different way from 
the file-system. Normally you would not access the object store directly from
within your programs but it is a good place to archive data to free up space for new calculations.
The object-store uses the same API as the Amazon S3 object store so many compatible clients and tools are available
 
 
+ Unlike files, objects cannot be modified or appended to. They are uploaded and downloade as complete objects.
  However it is possible to replace an Object with an entirely new version.
+ The Object store can be accessed from anywhere with an internet connection not just Cirrus.
 
Access Keys
===========

Object store access permissions and storage quotas are based on AccessKeys. An access key consists of two parts:

#. The name of the AccessKey
#. A corresponding AccessSecret

There is also a UUID that can be used as a unique identifier for the key

For example:

:AccessKey: AKIA74IP98S48W5D9EPR
:AccessSecret: xKvB5lXB3gbP47T+TLkRT+DUfl98BY20io0nkV9q
:UUID: 	3a36a0fef03518827ca41992e934850b8bbf7c28

These are a little bit like a randomly generated Username/Password pair which makes them difficult to guess but as you will need to store the secret in
tools and scripts care needs to be taken to ensure that they are kept secret.

Buckets
=======

Objects in the store are organised in collections called buckets. Every object has a URL of the form

https://cirrus-s3.epcc.ed.ac.uk/bucket-name/object-name

If an object is set to be "public" then *anyone* can download the object using a web-browser and this URL. For non-public objects additional parameters or http headers are needed to handle authentication.

Bucket and object names should therefore be chosen to ensure these URLs are valid. Good practice is to stick to alphanumeric characters underscores and hyphens. 
In particular you should avoid spaces as these cause problems with some tools. An object name *can* contain slashes giving the appearance of a directory structure within
the bucket. However this is purely cosmetic. File browsing tools usually present object names with slashes as a directory hierarchy but the object store just sees them as part of the object name.

Depending on the permissions the objects within a bucket may belong to different access-keys to the bucket itself. However storage quotas are always calculated based on the owner of the bucket not the object.
 
Permissions and ACLs
====================

Access permissions can be set on both buckets and objects. The Cirrus object store supports a combination of three permissions.

1) Read
2) Write
3) Full-control

For buckets:

+ Read permission allows you to lists its contents. You do *not* need this to access the object itself as long as you know the object name. 
+ Write permission allows you to add or delete objects.
+ Full-control allows you to change permissions on the bucket. 

For objects:

+ Read permission allows the object to be downloaded.
+ Full-control allows you to change permissions on the object.

These permissions can be granted to individual keys using an AccessControlList (ACL). You will usually need to specify the key using its UUID when doing this.
You can also grant permissions to two additional classes of users:

:Authenticated users: This means *any* active key known to the object store.
:All users: This means *everyone* with an internet connection.

You may want to set *read* permissions of this type when publishing *public* data but you should never grant write or full-control permissions to these groups.

The simplest way of using the object-store is for each user to have a personal access-key and quota. This will allow each user to store and retrieve their own data independently. However this 
makes large scale data sharing difficult. ACLs need to be set for every object that include the key of every user that needs access. Whenever membership of the access group changes, the ACLs of every object needs to be updated.

If you want to support data sharing you may wish to generate additional keys
representing different levels of access. For example a key that is allowed to modify a shared data-set and another that is only allowed to read it. You can then share these keys with the appropriate groups of people. This makes managing the ACLs much less work as they only have to reference the shared keys.
When someone leaves a group you can revoke their access by changing the AccessSecret for the shared key and re-distributing the new secret to the remaining members.



Managing the Object Store from SAFE
===================================

Keys and quotas are managed through the SAFE. If a SAFE project has an allocation on the object store there will be a "Object store quotas" section in the Project Administration page for that project.
Project managers can click on the button in that section to manage keys and quotas. From the quota management page you have two options:

New key
   To create a new AccessKey
List keys
   To show and manage existing keys.
   
When creating a key you need to provide a name for the key and a storage quota for the new key. The sum of all the key quotas within a project must be less than the total storage allocation of the project. Keys can be generated with a zero storage quota 
(for example to grant read-only access to a ashared data-set).

The List-keys page shows a list of the existing keys for the project. Click on one of the links to manage the corresponding key. The following options are available for each key:

View secret
   This shows details about the key including the AccessKey name, the AccessSecret and the UUID.
Set permissions
   This allows a project manager to share the key with selected members of the project. 
   When a key is shared with somebody they will be able to view and download the key from the SAFE. 
   If you want to revoke access to a key you can remove this permission then use *Regenerate* to change the AccessSecret. 
   Other people who still have the key shared with them 
   will be able to download the new secret as before.
Test
   The SAFE will connect to the object store using the key and check that the key is working.
List Buckets
   This shows the buckets owned by the key. 
   You can also click-through to the bucket and browse its contents (using that keys permissions). 
Change quota
   This allows a project manager to change the size of the storage quota allocated to the key.
Lock/Unlock
   An AccessKey can be locked/unlocked by a project manager. While a key is locked it cannot be used to access the object store.
Regenerate
   A project manger can use this to change the AccessSecret. 
   Permitted Users will be able to download the new value from the SAFE.


When a user had been given access to a key using the "Set permissions" menu the key will appear in their SAFE
navigation menu under "Login accounts"->"Credentials". This will then give them access to the following functions:

+ View secret
+ Test
+ List Buckets



Browsing the Object store from your desktop
===========================================

There are a number of File browser UIS that van be used to browse the object store on your desktop. For example the
Cloudberry browser https://www.cloudberrylab.com/explorer/amazon-s3.aspx.

+ Download and install the Freeware GUI from the above link.
+ Select File->"New S3 compatible account"->"S3 Compatible"
+ Fill in your AccessKey and AccessSecret. Use https://cirrus-s3.epcc.ed.ac.uk as the Service end-point.

Using a desktop GUI is usually the easiest way of creating and managing buckets.


Uploading and downloading Objects on Cirrus
==========================================

The Object store uses the Amazon S3 protocol so can be accessed using any of the standard tools developed to access AWS-S3.
For example *s3cmd*. To use s3cmd you need to first create a configuration file

+ run *s3cmd --configure*
+ Specify Access Key when prompted
+ Specify Secret Key when prompted
+ Default Region should be *uk-cirrus-1*
+ Specify *cirrus-s3.epcc.ed.ac.uk* when asked for the S3 endpoint.
+ Specify *cirrus-s3.epcc.ed.ac.uk/%(bucket)* for the next question.
+ Leave Encryption password blank.
+ Leave path to GPG unchanged.
+ Use HTTPS leave as *Yes*
+ Test the credential *Y* (default)
+ Select *y* to save the credential

You can re-run this command later to change any setting and it will default to your previous selection.

Run *s3cmd --help* to see the various supported commands. Though the Cirrus object-store does not support the CloudFront or Glacier options.
For example::
  -bash-4.1$ s3cmd mb s3://examplebucket
  Bucket 's3://examplebucket/' created
  -bash-4.1$ s3cmd put ~/random_2G.dat s3://examplebucket/random.dat
  WARNING: Module python-magic is not available. Guessing MIME types based on file extensions.
  upload: '/general/z01/z01/spb/random_2G.dat' -> 's3://examplebucket/random.dat'  [part 1 of 137, 15MB] [1 of 1]
   15728640 of 15728640   100% in    0s    22.16 MB/s  done
  upload: '/general/z01/z01/spb/random_2G.dat' -> 's3://examplebucket/random.dat'  [part 2 of 137, 15MB] [1 of 1]
   15728640 of 15728640   100% in    0s    25.31 MB/s  done

  ....

  upload: '/general/z01/z01/spb/random_2G.dat' -> 's3://examplebucket/random.dat'  [part 137 of 137, 8MB] [1 of 1]
   8388608 of 8388608   100% in    0s    32.80 MB/s  done
  -bash-4.1$ s3cmd ls s3://examplebucket
  2019-06-05 11:28 2147483648   s3://examplebucket/random.dat
