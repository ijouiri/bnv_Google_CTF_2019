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







































