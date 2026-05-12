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

