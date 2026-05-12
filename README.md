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

           
