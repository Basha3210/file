Kafka + Spark Streaming Example

Watch the video here
This is an example of building a Proof-of-concept for Kafka + Spark streaming from scratch. This is meant to be a resource for video tutorial I made, so it won't go into extreme detail on certain steps. It can still be used as a follow-along tutorial if you like.

Also, this isn't meant to explain the design of Kafka/Hadoop, instead it's an actual hands-on example. I'd recommend learning the basics of these technologies before jumping in.

When considering this POC, I thought Twitter would be a great source of streamed data, plus it would be easy to peform simple transformations on the data. So for this example, we will

create a stream of tweets that will be sent to a Kafka queue
pull the tweets from the Kafka cluster
calculate the character count and word count for each tweet
save this data to a Hive table
To do this, we are going to set up an environment that includes

a single-node Kafka cluster
a single-node Hadoop cluster
Hive and Spark
