# bnv_Google_CTF_2019
bnv was a web challenge from the recent google CTF the challenge says there's not much to see in this enterprise-ready web application 
and we're given a link to the challenge upon visiting the URL we are presented with a drop-down to pick one city out of three and we can 
submit and we get back some response which looks like some sort of an address if you look closely we also see Braley and also the title 
says blind Association is this a hint to some kind of a blind attack well let's see

![1](https://user-images.githubusercontent.com/44733558/83204188-21ca2c80-a143-11ea-96c3-61c6b86ba506.PNG)

let's look at its network requests using the Google's developer tools let's change this slashing and submit this again as you can see there was a post request made to an endpoint slash API slash search along with some post data
going with it the post data looks like Jason and has some message key and some number as its value

![2](https://user-images.githubusercontent.com/44733558/83205753-58a24180-a147-11ea-9659-1f1004682e51.PNG)

if you look at the response we can see that we got back some response data as well which also has some text which needs to be rendered onto the screen

![3](https://user-images.githubusercontent.com/44733558/83205932-b59df780-a147-11ea-8afe-cba3c26e27f4.PNG)

now that we know what's going on behind the scenes let's go ahead and try to mess around with the requests to see if there's anything weird about it

![4](https://user-images.githubusercontent.com/44733558/83205981-d403f300-a147-11ea-8309-49fe28cc8998.PNG)

for this I'll be using insomnia "https://insomnia.rest/" which is a handy API client this helps me with making requests and viewing responses very easily I usually use some sort of a proxy like BurpSuite of fiddler but for a change I'm gonna be using this let's create a new post request and name it whatever you like in this case we're gonna copy the URL from the dev tools and use it in the API client let's also copy the post Eider which was used by the application down here as well now that we have everything this looks fine let's go ahead and send the request and see if we'd get the same results 

![5](https://user-images.githubusercontent.com/44733558/83206339-8e93f580-a148-11ea-9496-403655aa96b3.PNG)

this point I thought this could be a no sequel injection and spent a good amount of time looking for them but later a friend of mine told me that it was an xxe so I quickly tested if the server did expect some XML data by simply changing the content type header from application Jason to application XML and sure enough I do get an error back but says start tag expected not found line 1 column 1

![6](https://user-images.githubusercontent.com/44733558/83206604-25f94880-a149-11ea-99c4-3a22aef46288.PNG)

so this basically means that it does expect some sort of an XML data so let's go ahead and try to see if there's any entity expansion  with the XML data 

![7](https://user-images.githubusercontent.com/44733558/83206836-ab7cf880-a149-11ea-890c-e54fe96e7587.PNG)

let's modify the JSON data to a valid XML string we can start by adding the XML declaration like this and we can wrap the entire numeric value inside the message tag which becomes the root node let's go ahead and send the request

![8](https://user-images.githubusercontent.com/44733558/83207027-26461380-a14a-11ea-8ddd-716bc03d3d48.PNG)

but we get a validation error it says that there is no DTD found okay fine let's add one inside the DTD have created a simple entity which is kind of like a variable and a reference date down below now let's go ahead and see what happens

![9](https://user-images.githubusercontent.com/44733558/83207214-82a93300-a14a-11ea-9975-977ab975679b.PNG)

hmm no declaration for the ailment message apparently the parser requires the defined element to be declared inside the DTD kind of like declaring a variable before even using it make sense so let's give it what it wants all we have to do is declare the message inside the DTD we can do that by specifying the element named message with the data definition type set to PC data now let's go ahead and try this again, and we get the correct response notice how the response is not Jason anymore seems like the xml parser reads our input processes it and gives us back the output

![10](https://user-images.githubusercontent.com/44733558/83207445-1f6bd080-a14b-11ea-8378-f30f0afb24e5.PNG)
 
additionally the entity expansions are allowed as you can clearly see that our identity did expand which means it worked now let's see if we can make a request from the inside to the outside world or simply a request to a website of our choice
  
![11](https://user-images.githubusercontent.com/44733558/83207485-3b6f7200-a14b-11ea-8133-c7bf5ee5deb6.PNG)
  
to test this i'll be using beesepta which is a simple service that gives you a subdomain and keeps a log of all the requests made to that website quick and simple
  
![12](https://user-images.githubusercontent.com/44733558/83207552-60fc7b80-a14b-11ea-976f-fe8d09fa8bb2.PNG)
  
so let's go ahead and add an external entity to make the egress request or simply an external request but it seems like it has failed to load the external entity,  it seems like we're not allowed to load any external DTD
    
![13](https://user-images.githubusercontent.com/44733558/83208869-c1d98300-a14e-11ea-8e60-bef526d963b9.PNG)
    
but let's go ahead and try to load a file let's say etc/passwrd and see if we can get a different error

![14](https://user-images.githubusercontent.com/44733558/83208519-d6694b80-a14d-11ea-9a7e-93fe6fa100ce.PNG)

and we do interestingly we get a markup error which means that the file was loaded correctly but since it was not well formed XML file it broke

![15](https://user-images.githubusercontent.com/44733558/83208530-db2dff80-a14d-11ea-92a2-00dcfad37a6f.PNG)

so now we have a way to enumerate the file names so let's see if we could find a flag file let's go ahead and try for /flag and we get back the same output which means that flag is in route and then flag as a file and all we have to do is somehow read it but how

![16](https://user-images.githubusercontent.com/44733558/83208533-dd905980-a14d-11ea-8b37-051a01cab09b.PNG)

since we're not allowed to load the external DTD can we load the internal ones are there internal ones if there were I think we can load them because we were able to load the etc password and it broke but that was for a different reason so now we need to find an internal file which is a valid XML file that can be included and some how get the flag

![17](https://user-images.githubusercontent.com/44733558/83208542-dff2b380-a14d-11ea-9564-7fc13027dfb6.PNG)

exploiting xxe with local DTD files essentially we can use the entities defined inside the local DTD file but we need to define it before loading it completely because the XML processor will pick the fast one which is the one that we have defined it if you want more information on how this works read the actual blog post "https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/"

![18](https://user-images.githubusercontent.com/44733558/83208547-e2550d80-a14d-11ea-9ecd-0ebe93c9d27c.PNG)

in the block-post is mentioned that the Linux box might have a DTD file located in the
/user/share/yelp /dtd/docbookx.dtd 
and this file has an entity named ISOasma regardless we can use this to write the DTD code inside of it now let's craft the DTD cod
 
![19](https://user-images.githubusercontent.com/44733558/83208551-e54ffe00-a14d-11ea-85b7-90506c11bf88.PNG)
 
since we know that the input what we sent out for the example was a wrong filename and we got back the same file name in the response as an error this can be abused first we read the contents of the file that we want it could be the flag file it could be the etc/password then we can try to read another file but this time we're going to make sure that the second false name is the contents of the first file that we just read obviously this will give us an error because there was no file which has the name as the contents of the first file in the error we get back the name of the file we try to read which means we also get back the contents of the first file hence arbitrary file read via xxe using local DTD
 
![20](https://user-images.githubusercontent.com/44733558/83208556-e719c180-a14d-11ea-9c00-dca835771dab.PNG)
 
let's try this stage one is to read the proper file which we want to let's say etc/passwrd
  
![21](https://user-images.githubusercontent.com/44733558/83208560-eaad4880-a14d-11ea-9b1a-46b0ebda01e9.PNG)
  
and now let's try to read another file whose name is the contents of their etc/password
  
![22](https://user-images.githubusercontent.com/44733558/83208567-eda83900-a14d-11ea-92e2-36351b1d9dd9.PNG)

now all that's left is to reference it down below finally we have to include DTD like this

![23](https://user-images.githubusercontent.com/44733558/83208573-f00a9300-a14d-11ea-877d-3e77ecce8aee.PNG)

one more thing since the DTD code has some XML breaking characters it's required to encode them like this

![24](https://user-images.githubusercontent.com/44733558/83208576-f3058380-a14d-11ea-854b-6aa19dbafe59.PNG)

awesome this looks fine let's go ahead and give this a shot

![25](https://user-images.githubusercontent.com/44733558/83208583-f567dd80-a14d-11ea-9b15-1eca90f6b64d.PNG)

and we got back the contents of the etc/passwrd now let's try to read the flag

![26](https://user-images.githubusercontent.com/44733558/83208587-f862ce00-a14d-11ea-9371-9ac36cc4612a.PNG)

the flag is in the root directory let's change the path to the flag and we can just remove the id entity since we know or need it now let's give this a shot and there we go we go back the flag

![27](https://user-images.githubusercontent.com/44733558/83208589-fac52800-a14d-11ea-869a-3170120bd16b.PNG)







































