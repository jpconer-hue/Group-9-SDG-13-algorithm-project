#inlcude <iostream>
#include <vector>
#include <string>
#include <queue>
#include <algorithm>
#include <map>

using namespace std;

const double INF = 1e9;

struct Edge {
    int way;
    double weight;
} ;

class Graph {
    public: 
           Graph () {}

void addEdge(int v1, int v2, double dist) {
m_graph[v1].push_back({v2, dist});
m_graph[v2].push_back({v1, dist});
 }   
void printGraph (const vector<string>& names) {
    for (const auto& element : m_graph) {
    cout << names[element.first]  << " connected to: " << endl;
     for (const auto& Evac : element.second) {
     cout << "  " << names[Evac.way] << " (" << Evac.weight << " km) \n";
 } 
 cout << "\n";
   }
  }
 void markFlooded(int u, int v, const vector<string>& names) { // Simulates flooded road
 cout << "[FLOOD] Closed road: " << names[u] << " <-> " << names[v] << endl; // Notify user
 auto& edges_u = m_graph[u]; // Get roads from u
 edges_u.erase( // Erase the road to v
 remove_if(edges_u.begin(), edges_u.end(), [v](Edge e){ return e.way == v; }), // Find edge where destination = v
            edges_u.end()
    );
 auto& edges_v = m_graph[v]; // Get roads from v
 edges_v.erase( // Erase the road to u
 remove_if(edges_v.begin(), edges_v.end(), [u](Edge e){ return e.way == u; }), // Find edge where destination = u
            edges_v.end()
        );
    }

const vector<Edge>& getNeighbors(int u) const { // Returns all roads from node u
 static const vector<Edge> empty; // Empty vector if node has no roads
 auto it = m_graph.find(u); // Check if node exists in map
 return it!= m_graph.end()? it->second : empty; // Return neighbors or empty
    } 

int size() const { return m_graph.size(); } // Returns number of nodes with roads

pair<vector<double>, vector<int>> dijkstraShortestPath(int start, int nodes) { // Core Dijkstra algorithm
 vector<double> distance(nodes, INF); // All distances start as infinity
 vector<int> parent(nodes, -1); // Parent array to reconstruct path
 priority_queue<pair<double, int>, vector<pair<double, int>>, greater<pair<double, int>>> pq; // Min-heap: {dist, node}         
 distance[start] = 0; // Distance to start is 0
 pq.push({0, start}); // Push start node to queue
 while (!pq.empty()) { // While there are nodes to process
 int u = pq.top().second; // Get node with smallest distance
 double d = pq.top().first; // Get that distance
 pq.pop(); // Remove from queue
 
 if (d > distance[u]) continue; // Skip if we already found better path
 
 if(m_graph.find(u) == m_graph.end()) continue; // Skip if node has no edges
 
 for(const auto& edge : m_graph.at(u)) { // Check all neighbors of u
 int v = edge.way; // Neighbor node
 double w = edge.weight; // Distance to neighbor
 
 if (v < nodes && distance[v] > distance[u] + w) { // If new path is shorter
 distance[v] = distance[u] + w; // Update distance
 parent[v] = u; // Set parent for path reconstruction 
 pq.push({distance[v], v}); // Push new distance to queue
     }
 }
}
 return {distance, parent}; // Return distances and parents
 }    

 private:
    map<int, vector<Edge>> m_graph; // Adjacency list: node -> list of roads
};

struct Map { // Wrapper for the whole Naga map
    vector<string> names; // Names of all locations
    Graph m_graph; // The actual graph of roads
    int coliseum; // ID of JMR Coliseum = evacuation center
    Map(int n) : coliseum(n) {} // Constructor sets coliseum ID
};

Map NagaMap() { // Builds the Naga City map
    Map m(7); // Coliseum is node 6, but we pass 7 for size
    m.names = {"Brgy. Sabang", "Barangay Lerma", "Brgy. Triangulo", "Panganiban Drive", "Barangay Tinago", "Brgy. Dayangdang", "JMR Coliseum"}; // All locations
    m.coliseum = 6; // JMR Coliseum is index 6
    m.m_graph.addEdge(0, 6, 1.8); // Sabang -> Coliseum: 1.8km
    m.m_graph.addEdge(1, 6, 0.5); // Lerma -> Coliseum: 0.5km
    m.m_graph.addEdge(2, 6, 1.0); // Triangulo -> Coliseum: 1.0km
    m.m_graph.addEdge(3, 6, 0.75); // Panganiban -> Coliseum: 0.75km
    m.m_graph.addEdge(4, 3, 0.1); // Tinago -> Panganiban: 0.1km
    m.m_graph.addEdge(5, 4, 0.65); // Dayangdang -> Tinago: 0.65km
    m.m_graph.addEdge(5, 2, 3.0); // Dayangdang -> Triangulo: 3.0km backup
    return m; // Return completed map
}

vector<int> getpath(const vector<int>& parent, int start, int end) { // Rebuilds route from parent array
    vector<int> path; // Empty path vector
    if (end >= parent.size() || (parent[end] == -1 && start!= end)) return path; // No path exists
    for (int v = end; v!= -1; v = parent[v]) { // Walk backwards from end to start
        path.push_back(v); // Add node to path
        if(v == start) break; // Stop when we reach start
    }
    reverse(path.begin(), path.end()); // Reverse to get start->end order
    return path; // Return final route
}

void route(const Map& m, const vector<int>& path, double totaldistance) { // Prints evacuation route
    if (path.empty() || totaldistance >= INF) { // If no path found
        cout << "The area is flooded. go to another route" << endl; // Error message
        return;
    }
    cout << "Safest Evacuation Route Going to Jesse M. Robredo Coliseum" << endl; // Header
    cout << "Total Distance: " << totaldistance << " km" << endl << endl; // Show distance
    for (int i = 0; i < path.size(); i++) { // Loop through each stop
        cout << i + 1 << " " << m.names[path[i]] << endl; // Print step number + location
    }
    cout << "Proceed to Jesse M. Robredo Coliseum now. Stay safe" << endl; // Final instruction
}

void runUnitTests() { // Tests to verify code works
  cout << "=== RUNNING UNIT TESTS ===\n";
  Graph g; // Test graph 1
  g.addEdge(0, 1, 5.0); // Add 0->1 road, 5km
  auto [dist, parent] = g.dijkstraShortestPath(0, 2); // Run Dijkstra
 assert(dist[1] == 5.0); // Check distance is correct
  cout << "[PASS] Test 1: addEdge works\n";
    
Graph g2; // Test graph 2
 g2.addEdge(0, 1, 1.0); // 0->1 = 1km
 g2.addEdge(1, 2, 1.0); // 1->2 = 1km 
g2.addEdge(0, 2, 2.5); // 0->2 = 2.5km direct
 auto [dist2, parent2] = g2.dijkstraShortestPath(0, 3); // Should pick 0->1->2 = 2km
 assert(dist2[2] == 2.0); // Verify shortest path used
assert(parent2[2] == 1); // Verify parent is correct
  cout << "[PASS] Test 2: dijkstra finds shortest path\n";
    
  Graph g3; // Test graph 3
 g3.addEdge(0, 1, 1.0); // Only 0->1 exists
 auto [dist3, parent3] = g3.dijkstraShortestPath(0, 3); // Node 2 is isolated
  assert(dist3[2] == INF); // Distance should be infinity
 vector<int> path = getpath(parent3, 0, 2); // Try to get path
   assert(path.empty()); // Path should be empty
  cout << "[PASS] Test 3: Handles isolated node / no path\n";
    
 Graph g4; // Test graph 4
  g4.addEdge(0, 1, 1.0); // 0->1
 g4.addEdge(1, 2, 1.0); // 1->2
  auto [dist4, parent4] = g4.dijkstraShortestPath(0, 3); // Run Dijkstra
 vector<int> path2 = getpath(parent4, 0, 2); // Reconstruct path
 assert(path2.size() == 3); // Should have 3 nodes
  assert(path2[0] == 0 && path2[1] == 1 && path2[2] == 2); // Check order
  cout << "[PASS] Test 4: getpath works\n";
 cout << "All tests passed!\n\n"; // All good
 }

 int main() { // Program starts here
    runUnitTests(); // Run tests first
    Map naga = NagaMap(); // Build Naga City map
    cout << "=== SDG 13 FLOOD EVACUATION ROUTING - NAGA CITY ===\n\n"; // Title
    cout << "Available locations:\n"; // Show menu
    for(int i = 0; i < naga.coliseum; i++) { // Loop through barangays only
        cout << i << ": " << naga.names[i] << "\n"; // Print ID + name
    }
    cout << "\n";
    int start; // User's location
    cout << "Enter your current location ID (0-" << naga.names.size()-2 << "): "; // Prompt
    cin >> start; // Get input
    if(start < 0 || start >= naga.coliseum) { // Validate input
        cout << "Invalid location ID.\n"; // Error
        return 1; // Exit
    }

 }
    cout << "\n[BASELINE] Calculating shortest safe path...\n"; // Status
    auto start_time = chrono::high_resolution_clock::now(); // Start timer
    auto [dist, parent] = naga.m_graph.dijkstraShortestPath(start, 7); // Run Dijkstra
    auto end_time = chrono::high_resolution_clock::now(); // End timer
    auto duration = chrono::duration_cast<chrono::microseconds>(end_time - start_time); // Get microseconds
    vector<int> path = getpath(parent, start, naga.coliseum); // Get route
    route(naga, path, dist[naga.coliseum]); // Print route
    cout << "\n[PROFILING] Dijkstra execution time: " << duration.count() << " microseconds\n"; // Show time
    cout << "[COMPLEXITY] Theoretical: O((V + E) log V) = O((6 + 7) log 6) ≈ 34 operations\n"; // Show complexity    
