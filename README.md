# MandarinPassCode
A simulation of the pass code system of china

There are some subsystems ( sub-services ):

## 1. Patient Front End
Simulate the app using by people.
## 2. Officer Front End 
Simulate the app using by the Doctor and police in white suit (MIW).
## 3. Seven Level Router
Find which app server serves which individual and routing to it. In this version, we simply use the hash of id to find it. In real business it could hold by a set of bloom filter and a set of kv NO-SQL. This routing service must start after all the app server (4). the app server could return the request by the routing service let them route to next possible server, this mechanism ensures the routing service could extend horizontally.
## 4. App And Data. 
since we never need to cross query across individual, it's very simple to extend the service horizontally. once a server starts, it read all trusted data from center database (5). once fully warmed, the server join the service to serve the query and modify the exist user's data and register a new patient. 

This service has 4 sub service:
1. individual data query. prepared string content the data Front Ends want inquire(1), (2). the query happens in memory.
2. individual data modify. modifying happens in memory then persistent and queued sync to center database (5)  
3. LostAndFound service, deal with known exist but can't found in any app server. this service must offer stronger service than (1) and (2) to avoid domino effect. This service have the power to order any app server to (queued) dump data back to center database and re-warm.
4. New Patient register.
5. Operation APIs. 
        
## 5. Center database
Holds trusted data. using traditional method to protect the service and the data.
## 6. Inter-system Sync 
...
## 7. Operation Interface, Analyzing and BI
... 
## 8. Gate
   To avoid attack, the gate check the requests and drop any unknown access. The gate service runs behind traditional anti-attack mechanism to offer 7 level filter.

# System Diagrams
## 1. Over View Diagram
ignoring the gate (8) in this diagram for a clear view, 
```mermaid
graph TD;
    PatientFrontEnd[Patient Front End]-->SevenLevelRouter{Seven Level Router};
    OfficerFrontEnd[Officer Front End]-->SevenLevelRouter{Seven Level Router};
    SevenLevelRouter-->AppAndData(App And Data);
    AppAndData-- return -.->SevenLevelRouter;
    AppAndData-- queued -->CenterDatabase[(Center Database)]
    CenterDatabase-->AppAndData
    CenterDatabase-- queued -->InterSystemSync(Inter System Sync)
    InterSystemSync-- queued -->CenterDatabase
    AnalyzingAndBI-->CenterDatabase
    Operation-->AppAndData
    Operation[operation interfaces]-->CenterDatabase
    Operation-->InterSystemSync

```