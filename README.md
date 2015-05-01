### conscientious-peers

Towards conscientious peers: combining agents and peers for efficient and scalable video segment retrieval for VoD services.

> In this project we provide the algorithms used to 'agentify' peers who belong to a P2P network and the source code implemented for a Video-on-Demand (VoD) environment, using the BitTorrent protocol concepts.

###Peer Algorithms

Update Resources States Algorithm.
```javascript
UPDATE_RESOURCES_STATES()
if isNotAlive(agent)
  findAgentInSwarm()
freeRiders <- agent.getFreeRidersInSwarm
ServiceModule.purgeConnections(freeRiders)
segmentsUp <- getUploadedSegments
segmentsDw <- getDownloadedSegments
agent.updateSegments(segmentsUp, segmentsDw, me)
bandwith <- obtainUploadBandwith
if bandwith < UPLOAD_LIMIT
  available <- true
else
  available <- false
agent.updateAvailableUpload(avail, me)
```

Peer Segment Discovery
```javascript
SEGMENT_DISCOVERY(Segment s)
if sequential(me)
  if ServiceModule.hasSegment(s) is true
    peers <- ServiceModule.nextPeers(s)
  else
    peers <- AgentModule.nextPeers(s)
  BUILD_SEQUENTIALLY (peers)
else
  peers <- AgentModule.jumpPeers(s)
  BUILD_JUMPS(peers)
```

Build Connections Using Sequential Policy
```javascript
BUILD_SEQUENTIALLY (List peers)
s <- LastDownloadedSegment
for each peer in peers
  if sequential(peer) and isAlive(peer)
    connect(me, peer)
    download(peer, s)
```

Build Connections Using Jump Policy
```javascript
BUILD_JUMP (List peers)
s <- LastDownloadedSegment
for each peer in peers
  if isAlive(peer)
    connect(me, peer)
    download(peer, s)
if size(peers) < LIMIT_CONNECTED_PEERS
  newPeers <- AgentModule.jumpPeers(s)
  BUILD_JUMP(newPeers)
```

###Agent Algorithms

Update Sequential Neighbors Algorithm
```javascript
UPDATE_NEIGHBORS()
segments <- getSwarmDownloadedSegments
members <- obtainSwarmMembers
if qty(members) < SWARM_LIMIT
  availJoin <- true
else
  availJoin <- false
bandwith <- obtainSwarmUploadBandwith
if bandwith < UPLOAD_LIMIT
  availUpload <- true
else
  availUpload <- false
for each agent in sequentialNeighborList
  if isAlive(agent)
    agent.updateResources(availJoin, availUpload, segments, agent)
  else
    sequentialNeighborsList.remove(agent)
newAgents <- sequentialNeighborsList.obtainSequentialNeighborsAgents
sequentialNeighborsList.add(newAgents)
```

Monitor Swarm Behaviour Algorithm
```javascript
MONITOR_SWARM ()
members <- obtainSwarmMembers
for each peer in members
  purge <- false
  if isNotAlive(peer)
    purge <- true
  if hasNotUploadSegment(peer)
    purge <- true
    untrustableList.add(peer)
  if purge is true
    members.remove(peer)
```

Process Join Swarm Algorithm
```javascript
PROCESS_JOIN (Peer p)
members <- obtainSwarmMembers
if qty(members) > SWARM_LIMIT
  returns false
for each agent in SequentialModule.sequentialNeighborsList
  option <- agent.isTrust(p)
  if option is TRUSTY
    join(p)
    returns true
  else if option is UNTRUSTY
    returns false
join(p)
returns true
```

Video Segments Search Algorithm
```javascript
SEGMENT_DISCOVERY (Segment s, Peer p, Boolean join)}
isMember <- SwarmModule.isMember (p)
if isMember is false and join is true
  SwarmModule.PROCESS_JOIN (p)
if segmentManagedBySwarm(s)
  peers <- SwarmModule.obtainAvailablePeers(s)
else if segmentManagedByNeighbors(s)
  peers <- SequentialModule.obtainAvailablePeers(s)
else if segmentManagedByJumps(s)
  peers <- GLOBAL_SEARCH (s, p, join)
returns peers
```

Video Segments Global Search Algorithm
```javascript
GLOBAL_SEARCH (Segment s, Peer p, Boolean wantJoins)
agentClose <- JumpingModule.obtainAgent(s)
if isSharing(agentClose, s)
  peers <- agentClose.obtainAvailablePeers(s)
else
  peers <- agentClose.SEGMENT_DISCOVERY(s, p, wantJoins)
returns peers
```

###Construction Algorithms

Peer-Agent Conversion Algorithm (in a first join)
```javascript
PROVIDES_VIDEO (Video v)
listAgents <- MASLayer.obtainAgents(v)
if listAgents is empty
  MASLayer.register(me)
else
  for each agent in listAgents
    SequentialModule.update(agent)
    JumpingModule.update(agent.joins)
```

Peer-Agent Conversion Algorithm
```javascript
CONVERT_TO_AGENT()
downloaded <- verifyDownloadAllSegments
if downloaded is true
  agent <- obtainAgentSwarm
  members <- agent.obtainSwarmMembers
  if qty(members) > SWARM_LIMIT
    SequentialModule.update($agent$)
    JumpingModule.update($agent$.joins)
    agent.removePeer(me)
    agent.addAgent(me)
```

Monitor Swarm Limits
```javascript
MONITOR_LIMITS()
members <- obtainSwarmMembers
if qty(members) > SWARM_LIMIT
  for each peer in members
    downloaded <- verifyDownloadAllSegments(peer)
    if downloaded is true
      agent.removePeer(peer)
      peer.CONVERT_TO_AGENT
```
