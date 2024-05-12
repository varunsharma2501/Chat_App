BACKEND
1.	DATABASE
•	Used mongoose model to connect to database and is asynchronous since connecting may take some time and we don’t want to block other operations
•	And then it is called in the server.js to make connection to db

2.	MODELs
1.	Conversations
•	Used to make a schema for conversations to be stored in database.
•	It has participants array which consists of users and a messages array which consists of messages as defined in its schema
•	Timestamp=true automatically adds the createdAt time in its schema
2.	Messages
•	Here also it contains a senderId and receiverId which are both objects of user schema and a message which is a string 
•	Timestamp =true again
3.	User
•	Contains fullname,username ,password,gender,profilePic
 3.UTILS
	A.GeneratreToken.js
•	const token = jwt.sign({ userId }, process.env.JWT_SECRET, {
•	        expiresIn: "15d",
•	    });
•	

jwt.sign(payload, secretOrPrivateKey, [options, callback])
•	This means that the structure of jwt is defined and in our case the userId is the payload or data which will help application recognize user once the token is decrypted
•	The jwt secret is used to decrypt the token
•	At last there are optional setting like here we set the expiry time

4.CONTROLLERS
A. Auth Controller
a.	SignUp functionality
•	First it fetches data from request like username,password,confirm password,gender
•	Then it encrypts the password using bcrypt
•	Then it creates a new User using the schema we defined in modals
•	Then user is created in database and to check if it is successful we check if(newUser) before making token
•	In webtoken we are using newUser._id and .ID attributes isn’t created by us but the backend creates it MongoDB automatically generates a unique identifier (_id) for each document inserted into a collection if one is not provided explicitly
•	So the server sends the token to client using the functionality in util we defined 

Similarly the login and logout can be understood

B. Message Controller
1.	Sendmessage
• We get the message ,receiverId,senderId from the request
• Then we find conversation using receiver and sender id
• Create a new message object and push to conversation and save both in db
• Get the receivers socket id and push the message to him
2.	getMessages
• get receiver and senderId from request
• find the conversation 
• use populate to get all conversations from db

C. User Controller
• This function retrieves a list of users (excluding the currently logged-in user) to display in the sidebar of the application.
• const filteredUsers = await User.find({ _id: { $ne: loggedInUserId } }).select("-password");: This line queries the database to find users whose IDs are not equal to the ID of the currently logged-in user.
$ne stands for "not equal to".
The .select("-password") part ensures that the password field is excluded from the retrieved user data. It's a common security practice not to send sensitive information like passwords over HTTP responses.
• res.status(200).json(filteredUsers);: This line sends a successful HTTP response (status code 200) containing the filtered user data in JSON format. The filtered users are the users other than the currently logged-in user.


5.SOCKET
• const io = new Server(server, {
cors: {
o	origin: ["http://localhost:3000"],
o	methods: ["GET", "POST"],
},
});
-The Server class in Socket.IO requires an existing HTTP server instance to attach to. This is because Socket.IO establishes WebSocket connections over HTTP initially before upgrading them to WebSocket connections. By passing the server parameter, Socket.IO knows which server instance to attach to.So we are not creating a separate server but we are just upgrading our existing server which we had to websocket
• so in the io.on(“connection” part what we are doing is once the client is connected i.e we open a browser then a socket gets connected to our main server.
• const userId = socket.handshake.query.userId;
this means that when connection is made then during that the initial query by that client to server is (socket.hadnshake.query) and in that parameter userId is passed and this is what we store in mapping
• in userSocketMap we have the map of all users till they are online and once offline the entry is deleted so it maintains the list of online users and emits it to all the clients

6.ROUTES
A. Auth routes
-used to get signup ,login and logout functionality

B.Message routes
-used for gerMessages and sendMessage functionality where we pass the receivers id as parameter

C.User Routes
-it is the default “/” path to get json of all users for sidebar
7.MIDDLEWARE
• This middleware is responsible for protecting routes by verifying the authenticity of JSON Web Tokens (JWTs) sent by clients.
• It extracts token from jwt and then verifies it using the jwt token and then matches the userId in db and fetches the user data and passes that users data to the req object

8.SERVER.JS
• Dotenv.config(): Loads environment variables from a .env file into process.env.
• app.use(express.static(path.join(__dirname, "/frontend/dist"))): Serves static files (like HTML, CSS, and JavaScript files) from the specified directory (frontend/dist).
• app.use(express.static(path.join(__dirname, "/frontend/dist"))): Serves static files (like HTML, CSS, and JavaScript files) from the specified directory (frontend/dist).
• app.get("*", ...): Handles all other routes. It sends the index.html file from the frontend/dist directory, allowing the frontend router (like Vue Router or React Router) to handle the routing on the client-side.Means that if we write any random url other than routes then we will just see index.html page
