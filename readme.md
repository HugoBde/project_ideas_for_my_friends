# polymarket market depth chart

## Websocket market channel api
To do a market depth chart, you will need to use the *websocket market channel* api (documented [here](https://docs.polymarket.com/developers/CLOB/websocket/market-channel). To use it, you need to know the token id that you're interested in.

## Events data API
To get the token_id for the market you're interested in, you need to use the *events* API specifically, the **get event by slug** endpoint, which is documented [here](https://docs.polymarket.com/api-reference/events/get-event-by-slug)

## What's the slug? How do I get a slug?
The slug is a human readable identification string for the market you're interested in. For bitcoin up-down 15 min, it's formatted like this: btc-updown-15m-<timestamp>. If you want to work on a different market, you can go find it on polymarket manually, and copy the end of the url. (E.g: https://polymarket.com/event/btc-updown-15m-1765748700 notice the end bit with the btc...). Note that an **event** can have multiple **markets**. Each **market** has a token pair, one token for the positive outcome and a token for the negative outcome.

## What's a timestamp? how do I make a timestamp?
to get the *timestamp* for the ongoing market, you need a little bit of computer science history. Back in the days, they needed to standardise the representation of time. Someone said: let's count the seconds since Jan 1st 1970. They (practically) all said yes. Midnight of Jan 1st 1970 is known as Unix time. history lesson over. The timestamp in the slug is the start time of the market, in seconds since unix time. I will give you the opportunity to experience the joy of programming in the pre-chatgpt era, by not telling you how to get the timestamp in python. You're allowed to google though. If you give up see below:
<details>
<summary>click here if you're gay</summary>
- import the datetime library (part of the python standard library so already installed on your computer)
- create a new datetime object that represents the time now using this function:
```python
right_now = datetime.datetime.now()
```
- get the unix time representation liek this:
``` python
timestamp = right_now().timestamp()
```
- realise that python gives you the timestamp as a floating point number which is useless for your use case, so convert it to an integer
``` python
timestamp = int(right_now().timestamp())
```
</details>
Note that each market starts a o'clock, quarter past, half past, quarter to. That means the timestamp should be a multiple of 15 minutes * 60 seconds = 900. I'll let you figure otu the math to floor your timestamp to the previous multiple of 900 to get the start time of the ongoing market

## Ok i have a timestamp now
congrats!!!
now put the slug together and hit that event api to get the event data. look at the documentation to find the *token_id* for the *market* you're interested in. it should be a pair, use the first one for the Up event. The down event is just the oppopsite of the up event anyway

## ok I have my token id now
congrats again!!
now read the the documentation on the websocket api to get live market data. 

## this is confusing
yes. maybe I should ahve said this earlier, but websocket is what is known as full-duplex. that means the server can send you data without you asking for it, unlike HTTP. quick little bit of background: your computer uses TCP to chat with the server. HTTP (P for protocol) dictates how to format the data transferred over TCP so that the server understands. HTTP is actually really well designed (which is a very rare thing in programming) in that it has lots of extensions. One is the ability to move from HTTP to anotehr protocol. The most common use of that is to swtich from HTTP (request response model) to WebSocket (full duplex, both sides speak whenever they want). 
Enough background, you'll need to check if the python requests library supports going into websocket mode. If it doesn't, find another library, re-learn how to use it and then continue onwards.
the full duplex model might make the way you write your code a little confusing, because you dont have this linear "i make a request and wait for a response" style. Isntead, you will probably have to write a function to handle messages from the server, and it will be called by the library. The library will probaly give you the opportunity to send some messages when the webscoket connection is established, which is when you subscribe to the market you're interested in. After that, you can just handle the emssages. 
The polymarke api has a few different type of messages. The most important ones are the book message, which gives you a snapshot of the current order book. this is when you copy the state so you can display it on screen. The other important msg is the price_change message, which happens whenever someone places a new order, or an order is cancelled. when you receive eitehr of those messages, you'll want to update your internal state basd on the update. If i remember correctly, if someone wants to buy a "UP" event, it's the equivalent of trying to sell a "DOWN", so those price udpate messages come in pair, which are the opposite of each other. Only proces the first one, but don't take my word for it.
