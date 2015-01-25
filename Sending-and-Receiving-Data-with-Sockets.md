# Overview

Network sockets are the endpoints of internet connections between devices. Basically we need two types of sockets to handle the connection - client and server. The main difference between them is that a server socket is listening for incoming connection requests. In this snippet I will try to show you a simple connection between an Android client device and a Java server app over a local network.

## Android client

We will be using AsyncTask to make connection as a background task, Callback Interface for managing the incoming messages, Handler for updating GUI and TCPClient class representing the client.
I will explain this on the Network Remote example - easy client-server app for shutting down the computer.

### AsyncTask

We are using AsyncTask to avoid StrictMode fatal error for network access ( Look in references ). The StrictMode policy is simply forbidding us to affect on UI Thread.

```java
/**
 * Created by Mariusz on 15.10.14.
 *
 * AsyncTask class which manages connection with server app and is sending shutdown command.
 */
public class ShutdownAsyncTask extends AsyncTask<String, String, TCPClient> {

    private static final String     COMMAND     = "shutdown -s"      ;
    private              TCPClient  tcpClient                        ;
    private              Handler    mHandler                         ;
    private static final String     TAG         = "ShutdownAsyncTask";

    /**
     * ShutdownAsyncTask constructor with handler passed as argument. The UI is updated via handler.
     * In doInBackground(...) method, the handler is passed to TCPClient object.
     * @param mHandler Handler object that is retrieved from MainActivity class and passed to TCPClient
     *                 class for sending messages and updating UI.
     */
    public ShutdownAsyncTask(Handler mHandler){
        this.mHandler = mHandler;
    }

    /**
     * Overriden method from AsyncTask class. There the TCPClient object is created.
     * @param params From MainActivity class empty string is passed.
     * @return TCPClient object for closing it in onPostExecute method.
     */
    @Override
    protected TCPClient doInBackground(String... params) {
        Log.d(TAG, "In do in background");

        try{
            tcpClient = new TCPClient(mHandler,
                                      COMMAND,
                                      "192.168.1.1",
                                      new TCPClient.MessageCallback() {
                @Override
                public void callbackMessageReceiver(String message) {
                    publishProgress(message);
                }
            });

        }catch (NullPointerException e){
            Log.d(TAG, "Caught null pointer exception");
            e.printStackTrace();
        }
        tcpClient.run();
        return null;
    }   
```	
In this AsyncTask we are creating TCPClient object ( explained below ). In the TCPClient constructor we are passing Handler object for changing the UI, 
COMMAND - the String with the "shutdown -s" command for shutting down the computer, IP number - the servers ip number; Callback object - when we are getting
servers response, the callback method 'messageCallbackReceiver' is starting 'publishProgress' method, which is publishing progress to 'onProgressUpdate' AsyncTask's
method.

```java
	/**
     * Overriden method from AsyncTask class. Here we're checking if server answered properly.
     * @param values If "restart" message came, the client is stopped and computer should be restarted.
     *               Otherwise "wrong" message is sent and 'Error' message is shown in UI.
     */
    @Override
    protected void onProgressUpdate(String... values) {
        super.onProgressUpdate(values);
        Log.d(TAG, "In progress update, values: " + values.toString());
        if(values[0].equals("shutdown")){
            tcpClient.sendMessage(COMMAND);
            tcpClient.stopClient();
            mHandler.sendEmptyMessageDelayed(MainActivity.SHUTDOWN, 2000);

        }else{
            tcpClient.sendMessage("wrong");
            mHandler.sendEmptyMessageDelayed(MainActivity.ERROR, 2000);
            tcpClient.stopClient();
        }
    }
```

After receiving proper message, we are sending command, or if we received wrong message, we are sending message "wrong" and stopping client.
After this we are being transferred to the 'onPostExecute' method:

```java
 @Override
    protected void onPostExecute(TCPClient result){
        super.onPostExecute(result);
        Log.d(TAG, "In on post execute");
        if(result != null && result.isRunning()){
            result.stopClient();
        }
        mHandler.sendEmptyMessageDelayed(MainActivity.SENT, 4000);

    }
}
```

 So step by step:
* AsyncTask is creating TCPClient object
* In TCPClient constructor we are passing Handler, Command, IP Number and Callback object
* When TCPClient begins connection, it sends message "shutdown" to the server
* When we are receiving message from server, the callback is passing it to 'onProgressUpdate'
* If the received message ( response from server ) is equal to "shutdown", we are sending COMMAND to server
* After sending it we are stopping client, which is transferring us to 'onPostExecute' method.
* Meanwhile, the handler is receiving empty messages with 'msg.what' integers defined in MainActivity, which are responsible for updating the GUI

Example of how the widget UI is updated
```java
mHandler = new Handler(){
            public void handleMessage(Message msg) {
                switch(msg.what){
                    case SHUTDOWN:
                        Log.d(mTag, "In Handler's shutdown");

                         views     = new RemoteViews(context.getPackageName(), R.layout.activity_main);
                         widget    = new ComponentName(context, MainActivity.class);
                         awManager = AppWidgetManager.getInstance(context);
                                     views.setTextViewText(R.id.state, "Shutting PC...");
                                     awManager.updateAppWidget(widget,views);
                        break;
```

###TCPClient

This class is responsible for maintaining the connection. I will explain it step by step:

In the first step we can see the objects passed from ShutdownAsyncTask and others. Additionally we can see the sendMessage and stopClient methods.

```java
public class TCPClient {

    private static final String            TAG             = "TCPClient"     ;
    private final        Handler           mHandler                          ;
    private              String            ipNumber, incomingMessage, command;
                         BufferedReader    in                                ;
                         PrintWriter       out                               ;
    private              MessageCallback   listener        = null            ;
    private              boolean           mRun            = false           ;


    /**
     * TCPClient class constructor, which is created in AsyncTasks after the button click.
     * @param mHandler Handler passed as an argument for updating the UI with sent messages
     * @param command  Command passed as an argument, e.g. "shutdown -r" for restarting computer
     * @param ipNumber String retrieved from IpGetter class that is looking for ip number.
     * @param listener Callback interface object
     */
    public TCPClient(Handler mHandler, String command, String ipNumber, MessageCallback listener) {
        this.listener         = listener;
        this.ipNumber         = ipNumber;
        this.command          = command ;
        this.mHandler         = mHandler;
    }

    /**
     * Public method for sending the message via OutputStream object.
     * @param message Message passed as an argument and sent via OutputStream object.
     */
    public void sendMessage(String message){
        if (out != null && !out.checkError()) {
            out.println(message);
            out.flush();
            mHandler.sendEmptyMessageDelayed(MainActivity.SENDING, 1000);
            Log.d(TAG, "Sent Message: " + message);

        }
    }

    /**
     * Public method for stopping the TCPClient object ( and finalizing it after that ) from AsyncTask
     */
    public void stopClient(){
        Log.d(TAG, "Client stopped!");
        mRun = false;
    }
```

The magic happens here - in 'run()' method. Here we are using 'try-catch' tools for handling exceptions ( server not enabled, ip not proper etc. ).
As you can see below, we have infinite while() loop for listening to the incoming messages. We can simply stop it and finalize with 'stopClient()' method ( used in ShutdownAsyncTask's 'onProgressUpdate' method)

```java
public void run() {

        mRun = true;

        try {
            // Creating InetAddress object from ipNumber passed via constructor from IpGetter class.
            InetAddress serverAddress = InetAddress.getByName(ipNumber);

            Log.d(TAG, "Connecting...");

            /**
             * Sending empty message with static int value from MainActivity
             * to update UI ( 'Connecting...' ).
             *
             * @see com.example.turnmeoff.MainActivity.CONNECTING
             */
            mHandler.sendEmptyMessageDelayed(MainActivity.CONNECTING,1000);

            /**
             * Here the socket is created with hardcoded port.
             * Also the port is given in IpGetter class.
             *
             * @see com.example.turnmeoff.IpGetter
             */
            Socket socket = new Socket(serverAddress, 4444);


            try {

                // Create PrintWriter object for sending messages to server.
                out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())), true);

                //Create BufferedReader object for receiving messages from server.
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));

                Log.d(TAG, "In/Out created");

                //Sending message with command specified by AsyncTask
                this.sendMessage(command);

                //
                mHandler.sendEmptyMessageDelayed(MainActivity.SENDING,2000);

                //Listen for the incoming messages while mRun = true
                while (mRun) {
                    incomingMessage = in.readLine();
                    if (incomingMessage != null && listener != null) {

                        /**
                         * Incoming message is passed to MessageCallback object.
                         * Next it is retrieved by AsyncTask and passed to onPublishProgress method.
                         *
                         */
                        listener.callbackMessageReceiver(incomingMessage);

                    }
                    incomingMessage = null;

                }

                Log.d(TAG, "Received Message: " +incomingMessage);

            } catch (Exception e) {

                Log.d(TAG, "Error", e);
                mHandler.sendEmptyMessageDelayed(MainActivity.ERROR, 2000);

            } finally {

                out.flush();
                out.close();
                in.close();
                socket.close();
                mHandler.sendEmptyMessageDelayed(MainActivity.SENT, 3000);
                Log.d(TAG, "Socket Closed");
            }

        } catch (Exception e) {

            Log.d(TAG, "Error", e);
            mHandler.sendEmptyMessageDelayed(MainActivity.ERROR, 2000);

        }

    }
```

The last thing in the client is the Callback interface. We have it in the TCPClient class in the end:

```java
/**
     * Callback Interface for sending received messages to 'onPublishProgress' method in AsyncTask.
     *
     */
    public interface MessageCallback {
        /**
         * Method overriden in AsyncTask 'doInBackground' method while creating the TCPClient object.
         * @param message Received message from server app.
         */
        public void callbackMessageReceiver(String message);
    }
```

## Android Server

### Soon! 

## References:

* [Strict Mode Policy](http://blog.vogella.com/2012/02/22/android-strictmode-networkonmainthreadexception/) - Vogella's blog post about this exception
* [Sockets](http://developer.android.com/reference/java/net/Socket.html) - Android Developers API about Sockets implementation in Android
* [TurnMeOffMobile](https://github.com/panwrona/TurnMeOffMobile) - The post was based on this app. Simple TCP Client as Android widget with connection handled by AsyncTasks + IP number finding dynamically.

Read about sockets generally on the [Oracle Sockets Tutorial](http://docs.oracle.com/javase/tutorial/networking/sockets/index.html).



### Socket Programming (Client)

We need a way to send data to a computer from our android device. Easiest way is given the local ip address of the computer on the network to simply send data via a socket:

* http://android-er.blogspot.com/2011/01/simple-communication-using.html - Guide to sending data via a socket
* http://stackoverflow.com/a/20131985/313399 - stackoverflow snippet which connects to a host, and can send data to it via an asynctask
* http://stackoverflow.com/questions/6309201/sending-tcp-data-from-android-as-client-no-data-being-sent - stackoverflow snippet sending data to an ip address via a socket
* http://thinkandroid.wordpress.com/2010/03/27/incorporating-socket-programming-into-your-applications/ (client piece is phone, server piece needs to be on computer)
* [TurnMeOffMobile](https://github.com/panwrona/TurnMeOffMobile) - Simple TCP Client as Android widget with connection handled by AsyncTasks + IP number finding dynamically.

## Socket Programming (Server)

To send messages between an Android phone and a computer, the computer needs to be running a program that listens for messages sent on the socket coming from the phone:

* http://tutorials.jenkov.com/java-networking/sockets.html
* http://www.journaldev.com/741/java-socket-server-client-read-write-example
* http://www.java2s.com/Code/Java/Network-Protocol/StringbasedcommunicationbetweenSocket.htm