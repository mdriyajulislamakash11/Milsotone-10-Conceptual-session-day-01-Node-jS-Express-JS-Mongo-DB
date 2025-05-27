MongoDB কোডের প্রতিটি লাইনের বিস্তারিত ব্যাখ্যা বাংলায় (বাংলা স্ক্রিপ্টে) দিলামঃ

js
Copy
Edit
const express = require('express');
🔹 Express লাইব্রেরি ইনপোর্ট করা হচ্ছে — এটি একটি জনপ্রিয় Node.js ফ্রেমওয়ার্ক যা দিয়ে REST API তৈরি করা সহজ হয়।

js
Copy
Edit
const cors = require('cors');
🔹 CORS (Cross-Origin Resource Sharing) ইনপোর্ট করা হচ্ছে, যাতে অন্য ডোমেইন থেকেও তোমার API রিকোয়েস্ট গ্রহণ করতে পারে।

js
Copy
Edit
const port = process.env.PORT || 5000;
🔹 port সেট করা হচ্ছে। যদি environment variable এ PORT দেওয়া থাকে, তাহলে সেটা ব্যবহার করবে, না হলে ৫০০০ ব্যবহার করবে।

js
Copy
Edit
const app = express();
🔹 Express অ্যাপ্লিকেশন তৈরি করা হচ্ছে। এর মধ্যেই সব API ডিফাইন করা হবে।

js
Copy
Edit
app.use(cors())
🔹 এই লাইনটি সব incoming request-এ CORS allow করে দিচ্ছে।

js
Copy
Edit
app.use(express.json());
🔹 এই লাইনটি দিয়ে JSON body parser যুক্ত করা হচ্ছে, যাতে request body থেকে JSON ডেটা নেওয়া যায়।

js
Copy
Edit
const { MongoClient, ServerApiVersion, ObjectId } = require('mongodb');
🔹 MongoDB-এর প্রয়োজনীয় ক্লাসগুলো (MongoClient, ServerApiVersion, ObjectId) ইনপোর্ট করা হচ্ছে।

js
Copy
Edit
const uri = "mongodb+srv://akashAhmed:zkGCearveUQRWIaG@cluster0.zchez.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0";
🔹 MongoDB অ্যাটলাস ডাটাবেইসের URI (connection string) এখানে সংরক্ষিত। এটা দিয়ে তোমার MongoDB-তে কানেক্ট করা হচ্ছে।

js
Copy
Edit
const client = new MongoClient(uri, {
serverApi: {
version: ServerApiVersion.v1,
strict: true,
deprecationErrors: true,
}
});
🔹 MongoDB Client তৈরি করা হচ্ছে। এটি দিয়ে ডাটাবেইসে কাজ করা হবে। এখানে serverApi দেওয়া হচ্ছে, যা MongoDB এর নতুন API ব্যবহারে সহায়তা করে।

run ফাংশন শুরু হচ্ছে:
js
Copy
Edit
async function run() {
try {
await client.connect();
🔹 MongoDB-তে কানেক্ট করা হচ্ছে। async হওয়ায় await ব্যবহার করা হয়েছে।

js
Copy
Edit
const userCollection = client.db("userInfo").collection("info");
🔹 userInfo নামের ডাটাবেইসের ভিতরে info নামের collection নেওয়া হয়েছে, যেখানে ইউজার ডেটা থাকবে।

✅ সমস্ত API গুলো এখন ডিফাইন করা হচ্ছে:
js
Copy
Edit
app.get("/users", async (req, res) => {
const cursor = userCollection.find()
const result = await cursor.toArray();
res.send(result)
})
🔹 /users এ GET রিকোয়েস্ট আসলে সব ইউজারের ডেটা ডাটাবেইস থেকে বের করে array আকারে পাঠিয়ে দেবে।

js
Copy
Edit
app.post("/users", async (req, res) => {
const user = req.body;
const result = await userCollection.insertOne(user);
res.send(result);
});
🔹 /users এ POST রিকোয়েস্ট আসলে req.body থেকে ইউজার ডেটা নিয়ে ডাটাবেইসে একটি নতুন ইউজার যুক্ত করা হবে।

js
Copy
Edit
app.get("/users/:id", async (req, res) =>{
const id = req.params.id;
const query = {\_id : new ObjectId(id)}
const users = await userCollection.findOne(query);
res.send(users)
})
🔹 /users/:id → ইউজারের ID দিয়ে একটি নির্দিষ্ট ইউজারের ডেটা বের করে পাঠিয়ে দেয়।

js
Copy
Edit
app.put("/users/:id", async (req, res) => {
const id = req.params.id;
const user = req.body;
const filter = {\_id : new ObjectId(id)}
const option = {upsert : true}
const updateUser = {
$set : {
name: user.name,
email: user.email,
}
}
const result = await userCollection.updateOne(filter, updateUser, option);
res.send(result)
})
🔹 /users/:id → PUT রিকোয়েস্ট আসলে একজন ইউজারের ডেটা আপডেট করে। upsert: true মানে, ইউজার না থাকলে নতুন করে ইনসার্টও করবে।

js
Copy
Edit
app.delete("/users/:id", async (req, res) =>{
const id = req.params.id;
const query = {\_id : new ObjectId(id)}
const result = await userCollection.deleteOne(query)
res.send(result)
})
🔹 /users/:id → DELETE রিকোয়েস্টে সেই \_id এর ইউজারকে ডাটাবেইস থেকে মুছে ফেলে।

js
Copy
Edit
await client.db("admin").command({ ping: 1 });
console.log("Pinged your deployment. You successfully connected to MongoDB!");
🔹 MongoDB কানেকশন সফল হয়েছে কিনা তা যাচাই করতে ping কমান্ড দেওয়া হয়েছে।

js
Copy
Edit
// await client.close();
🔹 এই লাইনটি ক্লায়েন্ট কানেকশন বন্ধ করার জন্য, কিন্তু এখানে কমেন্ট করে রাখা হয়েছে যেন সার্ভার বন্ধ না হয়।

সার্ভারের বেস রাউট:
js
Copy
Edit
app.get("/", (req, res) => {
res.send("server is connected")
});
🔹 রুট /-এ গেলে "server is connected" রেসপন্স দেখাবে। এটা সাধারণত সার্ভার চলছে কিনা যাচাই করতে ব্যবহৃত হয়।

js
Copy
Edit
app.listen(port, () => {
console.log(`server port is running... ${port}`)
});
🔹 Express সার্ভার চালু হচ্ছে এবং নির্দিষ্ট পোর্টে listen করছে। কনসোলে পোর্ট নাম দেখাবে।

🔐 Important Note (গুরুত্বপূর্ণ):
তোমার MongoDB URI-তে ইউজারনেম ও পাসওয়ার্ড খোলা আছে — এটি সিকিউর নয়। তুমি .env ফাইল ব্যবহার করে URI হাইড করতে পারো।

তুমি চাইলে আমি এটাকে .env ফাইলসহ secure করে সেটআপ করে দিতে পারি।

আরও জানতে চাও? কিভাবে ডাটাবেইস connect করো বা CRUD operation নিয়ে কাজ করো — সব বাংলায় বুঝিয়ে দিবো।
