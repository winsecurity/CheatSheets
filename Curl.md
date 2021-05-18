# cURL 

curl stands for commandline URL 
it can be used to get data , post data , move delete etc 

## GET Request

this is basic getting the webpage
```
curl https://example.com/webdav
```

## DELETE files

if we have delete access then we can delete file on webdav

```
curl -X DELETE https://example.com/webdav/file.txt
```

file.txt will be deleted

## PUT or uploading files

we can upload to webdav using curl PUT request

```
curl -X PUT https://example.com/webdav -d @test.txt
```
-d represents data 

## Create folder in WebDAV

```
curl -X MKCOL https://example.com/webdav/newfolder
```

## Uploading file to folder

```
curl -T 'filename' 'https://example.com/webdav/newfolder/'
```

## Authentication

if we have username and password we can use those in our curl request

```
curl -u 'username:password' https://example.com/webdav --basic

curl -u 'username:password' https://example.com/webdav --digest

curl -u 'username:password' https://example.com/webdav --anyauth

```

## MOVE files

```
curl -X MOVE 'Destination:https://example.com/webdav/newfilename' 'https://example.com/webdav/oldfile'
```

## See Response code
```
curl -X GET https://example.com/webdav -sm '%{http_code}'
```


