# BFS-traversal-with-GUI-in-java
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.Queue;

// Represents a node (vertex) in the graph
class Node {
    int x, y, id;        // Coordinates and unique ID for the node
    boolean visited;    // Flag to mark if the node has been visited
    Color color;        // Color used for visual representation

    Node(int x, int y, int id) {
        this.x = x;
        this.y = y;
        this.id = id;
        this.visited = false; // Initially, nodes are not visited
        this.color = Color.BLUE; // Default color for nodes
    }
}

// Represents an edge (connection) between two nodes
class Edge {
    Node from, to;   // The nodes connected by this edge
    Color color;     // Color used for visual representation

    Edge(Node from, Node to) {
        this.from = from;
        this.to = to;
        this.color = Color.BLACK; // Default color for edges
    }
}

// Main class that creates and manages the graph and BFS visualization
public class GraphBFS extends JPanel {
    private ArrayList<Node> nodes; // List of nodes in the graph
    private ArrayList<Edge> edges; // List of edges in the graph
    private Node selectedNode = null; // Currently selected node for edge creation
    private JTextArea logArea; // Text area for logging BFS steps

    // Constructor to set up the user interface
    public GraphBFS() {
        nodes = new ArrayList<>();
        edges = new ArrayList<>();

        setLayout(new BorderLayout());

        // Panel for drawing the graph
        JPanel graphPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                super.paintComponent(g);
                drawGraph(g); // Custom method to draw nodes and edges
            }

            private void drawGraph(Graphics g) {
                // Draw edges
                for (Edge edge : edges) {
                    g.setColor(edge.color);
                    g.drawLine(edge.from.x, edge.from.y, edge.to.x, edge.to.y);
                }

                // Draw nodes
                for (Node node : nodes) {
                    g.setColor(node.color);
                    g.fillOval(node.x - 15, node.y - 15, 30, 30); // Draw the node as a circle
                    g.setColor(Color.WHITE);
                    g.drawString(String.valueOf(node.id), node.x - 5, node.y + 5); // Draw the node ID in the center
                }
            }
        };
        graphPanel.setPreferredSize(new Dimension(600, 600));

        // Add mouse listener to handle node and edge creation
        graphPanel.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseClicked(MouseEvent e) {
                handleMouseClick(e);
            }
        });

        add(graphPanel, BorderLayout.CENTER);

        // Panel with button to start BFS
        JPanel controlPanel = new JPanel();
        JButton bfsButton = new JButton("Start BFS");
        bfsButton.addActionListener(e -> startBFS());
        controlPanel.add(bfsButton);

        add(controlPanel, BorderLayout.SOUTH);

        // Text area for logging BFS steps
        logArea = new JTextArea(10, 30);
        logArea.setEditable(false);
        JScrollPane scrollPane = new JScrollPane(logArea);
        add(scrollPane, BorderLayout.EAST);
    }

    // Handle mouse clicks for creating nodes and edges
    private void handleMouseClick(MouseEvent e) {
        if (e.getButton() == MouseEvent.BUTTON1) { // Left mouse button
            Node clickedNode = findNode(e.getX(), e.getY());

            if (selectedNode == null) { // If no node is selected
                if (clickedNode == null) { // No node at clicked position
                    createNewNode(e.getX(), e.getY()); // Create new node
                } else {
                    selectedNode = clickedNode; // Select existing node
                }
            } else { // If a node is already selected
                if (clickedNode != null && clickedNode != selectedNode) {
                    createEdge(selectedNode, clickedNode); // Create edge between selected node and clicked node
                } else {
                    selectedNode = null; // Deselect node if clicked node is the same
                }
            }
        }
    }

    // Create a new node at the specified coordinates
    private void createNewNode(int x, int y) {
        String input = JOptionPane.showInputDialog("Enter node ID:");
        try {
            int id = Integer.parseInt(input);
            nodes.add(new Node(x, y, id));
            repaint(); // Refresh the panel to show the new node
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(null, "Invalid input. Please enter a number.");
        }
    }

    // Create an edge between two nodes
    private void createEdge(Node from, Node to) {
        edges.add(new Edge(from, to));
        selectedNode = null; // Deselect node after creating the edge
        repaint(); // Refresh the panel to show the new edge
    }

    // Find a node at the specified coordinates
    private Node findNode(int x, int y) {
        for (Node node : nodes) {
            if (Math.sqrt(Math.pow(node.x - x, 2) + Math.pow(node.y - y, 2)) <= 30) {
                return node;
            }
        }
        return null; // No node found at the coordinates
    }

    // Start the BFS traversal from the specified source node
    private void startBFS() {
        if (nodes.isEmpty()) {
            JOptionPane.showMessageDialog(this, "No nodes to traverse.");
            return;
        }

        String input = JOptionPane.showInputDialog("Enter the source node ID for BFS:");
        int sourceId;

        try {
            sourceId = Integer.parseInt(input);
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(this, "Invalid input. Please enter a number.");
            return;
        }

        Node sourceNode = null;
        for (Node node : nodes) {
            if (node.id == sourceId) {
                sourceNode = node;
                break;
            }
        }

        if (sourceNode == null) {
            JOptionPane.showMessageDialog(this, "Source node not found.");
            return;
        }

        resetGraph();
        logArea.setText("");
        log("Starting BFS from node " + sourceNode.id);

        bfsVisit(sourceNode);

        log("BFS completed.");
    }

    // BFS traversal and visualization
    private void bfsVisit(Node sourceNode) {
        Queue<Node> queue = new LinkedList<>();
        queue.add(sourceNode);
        sourceNode.visited = true;
        sourceNode.color = Color.RED;
        repaint();

        while (!queue.isEmpty()) {
            Node currentNode = queue.poll();
            log("Visiting node " + currentNode.id);

            for (Edge edge : edges) {
                if (edge.from == currentNode && !edge.to.visited) {
                    edge.to.visited = true;
                    edge.to.color = Color.RED;
                    edge.color = Color.RED;
                    queue.add(edge.to);
                    repaint();
                    sleep(500); // Pause for a moment to visualize the traversal
                }
            }
        }
    }

    // Log BFS steps in the text area
    private void log(String message) {
        logArea.append(message + "\n");
    }

    // Reset the graph to its initial state
    private void resetGraph() {
        for (Node node : nodes) {
            node.visited = false;
            node.color = Color.BLUE;
        }

        for (Edge edge : edges) {
            edge.color = Color.BLACK;
        }

        repaint();
    }

    // Pause for visualization
    private void sleep(int milliseconds) {
        try {
            Thread.sleep(milliseconds);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    // Main method to run the application
    public static void main(String[] args) {
        JFrame frame = new JFrame("BFS Visualization");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(800, 600);
        frame.add(new GraphBFS());
        frame.setVisible(true);
    }
}
