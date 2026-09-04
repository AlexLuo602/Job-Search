---
question: "[[06.DesignTwitter]]"
topic:
  - Design
  - Heap
  - Hash Map
lc_difficulty: Medium
tags:
  - neetcode-150
attempt_date: 2026-08-02
my_difficulty: Hard
status: Redo (Too Long)
time_min: 70
num_mistakes: 2
review_concepts:
  - Heap
---
# Design Twitter

_Use a hash map of deques to cap recent tweets, and a K-way merge with a heap to generate the chronological news feed._

## My Approach

I designed the system around two hash maps: one mapping users to a set of their followees, and another mapping users to a `deque` of their tweets. To keep space bounded, I capped each user's tweet deque at 10 elements, since a single user can never mathematically contribute more than 10 posts to any given news feed. 

To track chronological order across the entire system, I maintained a global `clock` that incremented with every `postTweet` call. 

For `getNewsFeed`, I treated it exactly like merging $K$ sorted lists. I fetched the followee set (including the user themselves), grabbed the single most recent tweet from each, and pushed them into a max-heap using Python's `heapq` max-heap functions. Then, I popped the newest tweet, appended it to the result, and pushed that specific user's *next* most recent tweet into the heap, repeating until I had 10 tweets.

## Complexity

|**Complexity**|**Why**|
|---|---|
|Time O(K)|`postTweet`, `follow`, and `unfollow` are $O(1)$. `getNewsFeed` builds the initial heap in $O(K)$ by capping it at size 10 during the loop over followees (each push/pop is $O(\log 10) = O(1)$), then extracts the 10 newest tweets in $O(10 \log 10) = O(1)$ since the heap never exceeds size 10.|
|Space O(U * 10 + U * F)|$U$ users storing a strict maximum of 10 tweets each, plus the followee map where $F$ is the average number of followees per user.|

## Key Insight

The news feed generation is fundamentally a "merge K sorted lists" problem. By capping each user's stored tweets at 10 and using a global clock, you avoid searching through massive histories. A heap allows you to efficiently pull the absolute newest tweets from the pool of followed users without having to dump and sort everything they've ever posted.

## Mistakes / Gaps

1. **Self-follow state bug** — I tried to have the user follow themselves inside `postTweet`, but placed it inside an `if userId not in self.followee_map` block, meaning if they had followed someone *before* posting their first tweet, they'd never follow themselves.
```python
# Wrong
if userId not in self.followee_map:
    self.followee_map[userId] = set()
    self.followee_map[userId].add(userId)
```
```python
# Fixed
if userId not in self.followee_map:
    self.followee_map[userId] = set()
self.followee_map[userId].add(userId)
```
2. **Off-by-one feed size** — I used `<=` instead of `<` in the feed generation loop, causing the loop to run when `len(result)` was 10, appending an 11th tweet to the return array.
```python
# Wrong
while len(result) <= 10 and heap:
    ...
```
```python
# Fixed
while len(result) < 10 and heap:
    ...
```

## Code

```python
class Twitter:
    def __init__(self):
        self.tweets_map = dict()
        self.followee_map = dict()
        self.clock = 0

    def postTweet(self, userId: int, tweetId: int) -> None:
        if userId not in self.tweets_map:
            self.tweets_map[userId] = deque()

        self.tweets_map[userId].appendleft([self.clock, tweetId])

        if len(self.tweets_map[userId]) > 10:
            self.tweets_map[userId].pop()

        if userId not in self.followee_map:
            self.followee_map[userId] = set()
            
        self.followee_map[userId].add(userId)
        
        self.clock += 1

    def getNewsFeed(self, userId: int) -> List[int]:
        tweeters = set()
        heap = []
        result = []

        if userId in self.followee_map:
            tweeters = self.followee_map[userId]

        for tweeter in tweeters:
            if tweeter in self.tweets_map:
                tweet = self.tweets_map[tweeter][0]

                heapq.heappush(heap, [tweet[0], 0, tweeter])

                if len(heap) > 10:
                    heapq.heappop(heap)

        heapq.heapify_max(heap)

        while len(result) < 10 and heap:
            _, index, tweeter = heapq.heappop_max(heap)
            tweetId = self.tweets_map[tweeter][index][1]
            result.append(tweetId)

            index += 1
            if index < len(self.tweets_map[tweeter]):
                time = self.tweets_map[tweeter][index][0]
                heapq.heappush_max(heap, [time, index, tweeter])
        
        return result

    def follow(self, followerId: int, followeeId: int) -> None:
        if followerId not in self.followee_map:
            self.followee_map[followerId] = set()

        self.followee_map[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        if followerId not in self.followee_map or followerId == followeeId:
            return
        
        self.followee_map[followerId].discard(followeeId)
```

## Is My Solution Optimal?

Operations like posting and following should theoretically be instant ($O(1)$), and retrieving a feed requires at minimum looking at the head of every followee's tweet list ($O(K)$). My K-way merge hits this $O(K)$ time bound exactly, and limits space effectively by discarding irrelevant history. **Yes, optimal.**

## Code Improvements

- **`collections.defaultdict`** — Initializing `self.tweets_map = defaultdict(deque)` and `self.followee_map = defaultdict(set)` completely removes the need to manually check `if id not in map:` across all four methods.
- **Negative clock trick** — Instead of relying on Python's newly added internal `_max` heap functions, decrementing the clock (`self.clock -= 1`) makes the newest tweets the "smallest" numbers, naturally sorting them at the top of a standard Python Min-Heap.
- **Temporary self-follow** — Permanently adding the user to their own `followee_map` in `postTweet` muddies the system state. It's cleaner to temporarily combine them using `users = self.followee_map[userId] | {userId}` just for the duration of `getNewsFeed`.

## Best Solution

```python
class Twitter:
    def __init__(self):
        self.tweets = collections.defaultdict(deque)
        self.followees = collections.defaultdict(set)
        self.clock = 0

    def postTweet(self, userId: int, tweetId: int) -> None:
        # Negative clock naturally floats newest tweets to top of min-heap
        self.tweets[userId].appendleft((self.clock, tweetId))
        if len(self.tweets[userId]) > 10:
            self.tweets[userId].pop()
        self.clock -= 1

    def getNewsFeed(self, userId: int) -> List[int]:
        res = []
        heap = []
        
        # Temporarily combine user with followees for this query only
        users = self.followees[userId] | {userId}
        
        for u in users:
            if self.tweets[u]:
                time, tweetId = self.tweets[u][0]
                heapq.heappush(heap, (time, 0, u))
                
        while heap and len(res) < 10:
            time, idx, u = heapq.heappop(heap)
            res.append(self.tweets[u][idx][1])
            
            if idx + 1 < len(self.tweets[u]):
                next_time, _ = self.tweets[u][idx + 1]
                heapq.heappush(heap, (next_time, idx + 1, u))
                
        return res

    def follow(self, followerId: int, followeeId: int) -> None:
        self.followees[followerId].add(followeeId)

    def unfollow(self, followerId: int, followeeId: int) -> None:
        self.followees[followerId].discard(followeeId)
```

This version cleans up the state management and dictionary initializations, cutting the code length in half while preserving the same K-way merge algorithm and space bounds. It does trade away one optimization: the submitted code capped the heap at size 10 while building it (evicting the oldest head whenever it grew past 10), keeping every push/pop O(log 10) = O(1) for a true O(K) build. This version pushes all K heads uncapped, so the build is O(K log K) worst-case — still dominated by K for practical purposes, but not strictly identical complexity. Re-adding the same cap here would restore O(K) if that mattered more than the simpler code.