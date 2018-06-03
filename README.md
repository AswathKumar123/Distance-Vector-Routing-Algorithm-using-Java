# Distance-Vector-Routing-Algorithm-using-Java
We will implement a simplified version of the Distance Vector Routing Protocol. The protocol will  run on top of four servers/laptops (behaving as routers) using TCP or UDP. Each server runs on a machine at a pre-defined port number. The servers should be able to output their forwarding tables along with the cost and should be robust to link changes. A server should send out routing packets only in the following two conditions: a) periodic update and b) the user uses command asking for one. This is a little different from the original algorithm which immediately sends out update routing information when routing table changes.

1.	Getting Started


A Distance Vector Routing Algorithm.

2.	Protocol Specification


The various components of the protocol are explained step by step. Please strictly adhere to the specifications.

2.1	Topology Establishment

We will use four servers/computers/laptops. The four servers are required to form a network topology as shown in Fig. 1. Each server is supplied with a topology file at startup that it uses to build its initial routing table. The topology file is local and contains the link cost to the neighbors. For all other servers in the network, the initial cost would be infinity. Each server can only read the topology file for itself. The entries of a topology file are listed below:


•	<num-servers>
•	<num-neighbors>
•	<server-ID> <server-IP> <server-port>
•	<server-ID1> <server-ID2> <cost>


num-servers: total number of servers in the network.
num-neighbors: the number of directly linked neighbors of the server.
server-ID, server-ID1, server-ID2: a unique identifier for a server, which is assigned by you.
cost: cost of a given link between a pair of servers. Assume that cost is an integer value.
 
Here is an example, consider the topology in Figure 1. We give a topology file for server 1 as shown in the table below.

4

5	1



4
1	3




7	2

2


Figure 1: The network topology


Line number	Line entry	Comments
1	4	number of servers
2	3	number of edges or neighbors
3	1 128.205.36.8 4091	server-id 1 and corresponding IP, port pair
4	2 128.205.35.24 4094	server-id 2 and corresponding IP, port pair
5	3 128.205.36.24 4096	server-id 3 and corresponding IP, port pair
6	4 128.205.36.4 7091	server-id 4 and corresponding IP, port pair
8	1 2 7	server-id and neighbor id and cost
9	1 3 4	server-id and neighbor id and cost
10	1 4 5	server-id and neighbor and cost

Your topology files should only contain the Line entry part. In each line, every two elements (e.g., server-id and corresponding IP, corresponding IP and port number) should be separated with a space. For cost values, each topology file should only contain the cost values of the host server’s neighbors (The host server here is the one which will read this topology file). Note that the IPs of servers may change when you are running the program in a wireless network environment. So, we need to use ifconfig or ipconfig to obtain the IP first and then set up the topology file before the demo.


IMPORTANT: In this environment, costs are bi-directional i.e. the cost of a link from A-B is the same for B-A. Whenever a new server is added to the network, it will read its topology file to determine who its neighbors are. Routing updates are exchanged periodically between neighboring servers. When this newly added server sends routing messages to its neighbors, they will add an entry
 
in their routing tables corresponding to it. Servers can also be removed from a network. When a server has been removed from a network, it will no longer send distance vector updates to its neighbors. When a server no longer receives distance vector updates from its neighbor for three consecutive update intervals, it assumes that the neighbor no longer exists in the network and makes the appropriate changes to its routing table (link cost to this neighbor will now be set to infinity but not remove it from the table). This information is propagated to other servers in the network with the exchange of routing updates. Please note that although a server might be specified as a neighbor with a valid link cost in the topology file, the absence of three consecutive routing updates from this server will imply that it is no longer present in the network.

2.2	Routing Update

Routing updates are exchanged periodically between neighboring servers based on a time interval specified at the startup. In addition to exchanging distance vector updates, servers must also be able to respond to user-specified events. There are 4 possible events in this system. They can be grouped into three classes: topology changes, queries and exchange commands: (1) Topology changes refer to an updating of link status (update). (2) Queries include the ability to ask a server for its current routing table (display), and to ask a server for the number of distance vectors it has received (packets). In the case of the packets command, the value is reset to zero by a server after it satisfies the query. (3) Exchange commands can cause a server to send distance vectors to its neighbors immediately.

2.3	Message Format

Routing updates are sent using the General Message format. All routing updates are UDP unreliable messages. The message format for the data part is:


0	1	2	3	(10 bits)

0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 (bit)

Number of update fields	Server port
Server IP
Server IP address 1
Server port 1	0x0
Server ID 1	Cost 1
Server IP address 2
Server port 2	0x0
Server ID 2	Cost 2
….

•	Number of update fields: (2 bytes): Indicate the number of entries that follow.

•	Server port: (2 bytes) port of the server sending this packet.

•	Server IP: (4 bytes) IP of the server sending this packet.
 
•	Server IP address n: (4 bytes) IP of the n-th server in its routing table.

•	Server port n: (2 bytes) port of the n-th server in its routing table.

•	Server ID n: (2 bytes) server id of the n-th server on the network.

•	Cost n: cost of the path from the server sending the update to the n-th server whose ID is given in the packet.
Note:
First, the servers listed in the packet can be any order i.e., 5, 3, 2, 1, 4. Second, the packet needs to include an entry to reach itself with cost 0 i.e. server 1 needs to have an entry of cost 0 to reach server 1.

3.	Server Commands/Input Format

The server must support the following command at startup:
•	server -t <topology-file-name> -i <routing-update-interval>

topology-file-name: The topology file contains the initial topology configuration for the server, e.g., timberlake_init.txt. Please adhere to the format described in 3.1 for your topology files.
routing-update-interval: It specifies the time interval between routing updates in seconds.  port and server-id: They are written in the topology file. The server should find its port and server-id in the topology file without changing the entry format or adding any new entries.


The following commands can be specified at any point during the run of the server:
•	update <server-ID1> <server-ID2> <Link Cost>

server-ID1, server-ID2: The link for which the cost is being updated.
Link Cost: It specifies the new link cost between the source and the destination server. Note that this command will be issued to both server-ID1 and server-ID2 and involve them to update the cost and no other server.
For example:
update 1 2 inf: The link between the servers with IDs 1 and 2 is assigned to infinity.
update 1 2 8: Change the cost of the link to 8.


•	step

Send routing update to neighbors right away. Note that except this, routing updates only happen periodically.


•	packets

Display the  number  of  distance  vector  packets  this  server  has  received  since  the  last invocation of this information.


•	display

Display the current routing table. And the table should be displayed in a sorted order from
 
small ID to big. The display should be formatted as a sequence of lines, with each line indicating: <destination-server-ID> <next-hop-server-ID> <cost-of-path>


•	disable <server-ID>

Disable the link to a given server. Doing this “closes” the connection to a given server with
server-ID. Here you need to check if the given server is its neighbor.


•	crash

“Close” all connections. This is to simulate server crashes. Close all connections on all links. The neighboring servers must handle this close correctly and set the link cost to infinity.



4.	Server Responses/Output Format

The following are a list of possible responses a user can receive from a server:
•	On successful execution of an update, step, packets, display or disable command, the server must display the following message:

<command-string> SUCCESS

where command-string is the command executed. Additional output as desired (e.g., for display, packets, etc. commands) is specified in the previous section.
•	Upon encountering an error during execution of one of these commands, the server must display the following response:

<command-string> <error message>

where error message is a brief description of the error encountered.
•	On successfully receiving a route update message from neighbors, the server must display the following response:

RECEIVED A MESSAGE FROM SERVER <server-ID>

where the server-ID is the id of the server which sent a route update message to the local server.

5.	Program

package prakash.ram.model;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.net.Inet4Address;
import java.net.InetAddress;
import java.net.InetSocketAddress;
import java.net.NetworkInterface;
import java.net.SocketException;
import java.nio.ByteBuffer;
import java.nio.channels.SelectionKey;
import java.nio.channels.Selector;
import java.nio.channels.SocketChannel;
import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.HashSet;
import java.util.Iterator;
import java.util.List;
import java.util.Map;
import java.util.Scanner;
import java.util.Set;
import java.util.Timer;
import java.util.TimerTask;

import com.fasterxml.jackson.databind.ObjectMapper;

import prakash.ram.client.Client;
import prakash.ram.server.Server;
import prakash.ram.utilies.TableBuilder;

public class dv {
	
	
	static int time;
	
	public static List<SocketChannel> openChannels = new ArrayList<>();
	public static Selector read;
	public static Selector write;
	static String myIP = "";
	static int myID = Integer.MIN_VALUE+2;
	public static Node myNode = null;
	//public static Map<Node,Integer> routingTable = new HashMap<Node,Integer>();
	public static List<Node> nodes = new ArrayList<Node>();
	public static List<String> routingTableMessage = new ArrayList<String>();
	public static Map<Node,Integer> routingTable = new HashMap<Node,Integer>();
	public static Set<Node> neighbors = new HashSet<Node>();
	public static int numberOfPacketsReceived = 0;
	public static Map<Node,Node> nextHop = new HashMap<Node,Node>();
 	public static void main(String[] args) throws IOException{
		
		read = Selector.open();
		write = Selector.open();
		Server server = new Server(2000);
		server.start();
		System.out.println("Server started running...");
		Client client = new Client();
		client.start();
		System.out.println("Client started running...");
		myIP = getMyLanIP();
		
		Timer timer = new Timer();
		Scanner in = new Scanner(System.in);
		boolean run = true;
		boolean serverCommandInput = false;
		while(run) {
			System.out.println("\n");
			System.out.println("*********Distance Vector Routing Protocol**********");
			System.out.println("Help Menu");
			System.out.println("--> Commands you can use");
			System.out.println("1. server <topology-file> -i <time-interval-in-seconds>");
			System.out.println("2. update <server-id1> <server-id2> <new-cost>");
			System.out.println("3. step");
			System.out.println("4. display");
			System.out.println("5. disable <server-id>");
			System.out.println("6. crash");
			String line = in.nextLine();
			String[] arguments = line.split(" ");
			String command = arguments[0];
			switch(command) {
			case "server": //server <topology-file-name> -i <routing-update-interval>
				if(arguments.length!=4){
					System.out.println("Incorrect command. Please try again.");
					break;
				}
				try{
				if(Integer.parseInt(arguments[3])<15){
					System.out.println("Please input routing update interval above 15 seconds.");
				}
				}catch(NumberFormatException nfe){
					System.out.println("Please input an integer for routing update interval.");
					break;
				}
				if((arguments[1]=="" || arguments[2]=="" || !arguments[2].equals("-i") || arguments[3]=="")){
					System.out.println("Incorrect command. Please try again.");
					break;
				}
				else{
					serverCommandInput = true;
					String filename = arguments[1];
					time = Integer.parseInt(arguments[3]);
					readTopology(filename);
					timer.scheduleAtFixedRate(new TimerTask(){
						@Override
						public void run() {
						try {
							step();
						} catch (IOException e) {
							e.printStackTrace();
						}
						}
						}, time*1000, time*1000);
				}
				break;
			case "update": //update <server-id1> <server-id2> <link Cost>
				if(serverCommandInput)
					update(Integer.parseInt(arguments[1]),Integer.parseInt(arguments[2]),Integer.parseInt(arguments[3]));
				else
					System.out.println("Please input the server command. Thank you.");
				break;
			case "step":
				if(serverCommandInput)
					step();
				else
					System.out.println("Please input the server command. Thank you.");
				break;
			case "packets":
				if(serverCommandInput)
					System.out.println("Number of packets received yet = "+numberOfPacketsReceived);
				else
					System.out.println("Please input the server command. Thank you.");
				break;
			case "display":
				if(serverCommandInput)
					display();
				else
					System.out.println("Please input the server command. Thank you.");
				break;
			case "disable":
				if(serverCommandInput){
					int id = Integer.parseInt(arguments[1]);
					Node disableServer = getNodeById(id);
					disable(disableServer);
				}
				else
					System.out.println("Please input the server command. Thank you.");
				break;
			case "crash":
				if(serverCommandInput){
					run = false;
					for(Node eachNeighbor:neighbors){
						disable(eachNeighbor);
					}
					System.out.println("Bubyee!! Thank you.");
					timer.cancel();
					System.exit(1);
				}
				else
					System.out.println("Please input the server command. Thank you.");
				break;
				default:
					System.out.println("Wrong command! Please check again.");
			}
		}
		in.close();
	}
	
	private static String getMyLanIP() {
		try {
		    Enumeration<NetworkInterface> interfaces = NetworkInterface.getNetworkInterfaces();
		    while (interfaces.hasMoreElements()) {
		        NetworkInterface iface = interfaces.nextElement();
		        if (iface.isLoopback() || !iface.isUp() || iface.isVirtual() || iface.isPointToPoint())
		            continue;

		        Enumeration<InetAddress> addresses = iface.getInetAddresses();
		        while(addresses.hasMoreElements()) {
		            InetAddress addr = addresses.nextElement();

		            final String ip = addr.getHostAddress();
		            if(Inet4Address.class == addr.getClass()) return ip;
		        }
		    }
		} catch (SocketException e) {
		    throw new RuntimeException(e);
		}
		return null;
	}

	public static void readTopology(String filename) {
		File file = new File("distance-vector/src/"+filename);
		try {
			Scanner scanner = new Scanner(file);
			int numberOfServers = scanner.nextInt();
			int numberOfNeighbors = scanner.nextInt();
			scanner.nextLine();
			for(int i = 0 ; i < numberOfServers;i++) {
				String line = scanner.nextLine();
				String[] parts = line.split(" ");
				Node node = new Node(Integer.parseInt(parts[0]),parts[1],Integer.parseInt(parts[2]));
				nodes.add(node);
				int cost = Integer.MAX_VALUE-2;
				if(parts[1].equals(myIP)) {
					myID = Integer.parseInt(parts[0]);
					myNode = node;
					cost = 0;
					nextHop.put(node, myNode);
				}
				else{
					nextHop.put(node, null);
				}
				routingTable.put(node,cost);
				connect(parts[1], Integer.parseInt(parts[2]),myID);
			}
			for(int i = 0 ; i < numberOfNeighbors;i++) {
				String line = scanner.nextLine();
				String[] parts = line.split(" ");
				int fromID = Integer.parseInt(parts[0]);int toID = Integer.parseInt(parts[1]); int cost = Integer.parseInt(parts[2]);
				if(fromID == myID){
					Node to = getNodeById(toID);
					routingTable.put(to, cost);
					neighbors.add(to);
					nextHop.put(to, to);
				}
				if(toID == myID){
					Node from = getNodeById(fromID);
					routingTable.put(from, cost);
					neighbors.add(from);
					nextHop.put(from, from);
				}
			}
			System.out.println("Reading topology done.");
			scanner.close();
		} catch (FileNotFoundException e) {
			System.out.println(file.getAbsolutePath()+" not found.");
		}
        
	}
	
	
	public static Node getNodeById(int id){
		for(Node node:nodes) {
			if(node.getId() == id) {
				return node;
			}
		}
		return null;
	}
	public static void update(int serverId1, int serverId2, int cost) throws IOException {
		if(serverId1 == myID){
			Node to = getNodeById(serverId2);
			if(isNeighbor(to)){
				routingTable.put(to, cost);
				Message message = new Message(myNode.getId(),myNode.getIpAddress(),myNode.getPort(),"update");
				message.setRoutingTable(makeMessage());
				sendMessage(to,message);
				System.out.println("Message sent to "+to.getIpAddress());
				System.out.println("Update success");
			}
			else{
				System.out.println("You can only update cost to your own neigbour!");
			}
		}
		if(serverId2 == myID){
			Node to = getNodeById(serverId1);
			if(isNeighbor(to)){
				routingTable.put(to, cost);
				Message message = new Message(myNode.getId(),myNode.getIpAddress(),myNode.getPort(),"update");
				message.setRoutingTable(makeMessage());
				sendMessage(to,message);
				System.out.println("Message sent to "+to.getIpAddress());
				System.out.println("Update success");
			}
			else{
				System.out.println("You can only update cost to your own neigbour!");
			}
		}
	}
	
	public static boolean isNeighbor(Node server){
		if(neighbors.contains(server))
			return true;
		return false;
	}
	public static List<String> makeMessage(){
		List<String> message = new ArrayList<String>();
		for (Map.Entry<Node, Integer> entry : routingTable.entrySet()) {
		    Node key = entry.getKey();
		    Integer value = entry.getValue();
		    message.add(key.getId()+"#"+value);
		}
		return message;
	}
	public static void connect(String ip, int port, int id) {
		System.out.println("Connecting to ip:- "+ip);
		try {
			if(!ip.equals(myIP)) {

				SocketChannel socketChannel = SocketChannel.open();
				socketChannel.connect(new InetSocketAddress(ip,port));
				socketChannel.configureBlocking(false);
				socketChannel.register(read, SelectionKey.OP_READ);
				socketChannel.register(write,SelectionKey.OP_WRITE);
				openChannels.add(socketChannel);
				System.out.println(".......");
				System.out.println("Connected to "+ip);
			}
			else {
				System.out.println("You cannot connect to yourself!!!");
			}
		}catch(Exception e) {
			e.printStackTrace();
		}
	}
	
	
	public static Node getNodeByIP(String ipAddress){
		for(Node node:nodes){
			if(node.getIpAddress().equals(ipAddress)){
				return node;
			}
		}
		return null;
	}
	public static void step() throws IOException{
		if(neighbors.size()>=1){
			Message message = new Message(myNode.getId(),myNode.getIpAddress(),myNode.getPort(),"step");
			message.setRoutingTable(makeMessage());
			for(Node eachNeighbor:neighbors) {
				sendMessage(eachNeighbor,message); //sending message to each neighbor
				System.out.println("Message sent to "+eachNeighbor.getIpAddress()+"!");
			}
			System.out.println("Step SUCCESS");
		}
		else{
			System.out.println("Sorry. No neighbors found to execute the step command.");
		}
		
	}
	
	public static void sendMessage(Node eachNeighbor, Message message) throws IOException{
		int semaphore = 0;
		try {
			semaphore = write.select();
			if(semaphore>0) {
				Set<SelectionKey> keys = write.selectedKeys();
				Iterator<SelectionKey> selectedKeysIterator = keys.iterator();
				ByteBuffer buffer = ByteBuffer.allocate(5000);
				ObjectMapper mapper = new ObjectMapper();
				String msg = mapper.writeValueAsString(message);
				
				buffer.put(msg.getBytes());
				buffer.flip();
				while(selectedKeysIterator.hasNext())
				{
					SelectionKey selectionKey=selectedKeysIterator.next();
					if(parseChannelIp((SocketChannel)selectionKey.channel()).equals(eachNeighbor.getIpAddress()))
					{
						SocketChannel socketChannel=(SocketChannel)selectionKey.channel();
						socketChannel.write(buffer);
					}
					selectedKeysIterator.remove();
				}
			}
		}catch(Exception e) {
			System.out.println("Sending failed because "+e.getMessage());
		}
	}
	
	public static String parseChannelIp(SocketChannel channel){//parse the ip form the SocketChannel.getRemoteAddress();
		String ip = null;
		String rawIp =null;  
		try {
			rawIp = channel.getRemoteAddress().toString().split(":")[0];
			ip = rawIp.substring(1, rawIp.length());
		} catch (IOException e) {
			System.out.println("can't convert channel to ip");
		}
		return ip;
	}
	
	public static Integer parseChannelPort(SocketChannel channel){//parse the ip form the SocketChannel.getRemoteAddress();
		String port =null;  
		try {
			port = channel.getRemoteAddress().toString().split(":")[1];
		} catch (IOException e) {
			System.out.println("can't convert channel to ip");
		}
		return Integer.parseInt(port);
	}
	
	public static boolean disable(Node server) throws IOException{
		if(isNeighbor(server)){
			
			sendMessage(server,new Message(myNode.getId(),myNode.getIpAddress(),myNode.getPort(),"disable"));
			for(SocketChannel channel:openChannels){
				if(server.getIpAddress().equals(parseChannelIp(channel))){
					try {
						channel.close();
					} catch (IOException e) {
						System.out.println("Cannot close the connection;");
					}
					openChannels.remove(channel);
					break;
				}
			}
			routingTable.put(server,Integer.MAX_VALUE-2);
			neighbors.remove(server);
			System.out.println("Disabled connection with server "+server.getId()+"("+server.getIpAddress()+")");
			return true;
		}
		else{
			System.out.println("You can only disable connection with your neighbor!!");
			return false;
		}
	}
	public static void display() {
		TableBuilder tb = new TableBuilder();
		tb.addRow("Destination Server ID","Next Hop Server ID","Cost");
		Collections.sort(nodes,new NodeComparator());
		for(Node eachNode:nodes){
			int cost = routingTable.get(eachNode);
			String costStr = ""+cost;
			if(cost==Integer.MAX_VALUE-2){
				costStr = "infinity";
			}
			tb.addRow(""+eachNode.getId(),""+nextHop.get(eachNode).getId(),costStr);
		}
		System.out.println(tb.toString());
	}
	
	
}

class NodeComparator implements Comparator<Node> {
    @Override
    public int compare(Node n1, Node n2) {
    	Integer id1 = n1.getId();
    	Integer id2 = n2.getId();
        return id1.compareTo(id2);
    }
}


