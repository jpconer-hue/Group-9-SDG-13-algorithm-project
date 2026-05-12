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
 
           
