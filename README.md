## spark3D

spark3D is as an open-source Apache Spark extension to process large-scale 3D data-sets from, Astrophysics, High Energy Physics, Meteorology, etc. The library works on top of the latest Spark versions (2.0+), has user-friendly APIs (in Scala and Python), works on top of any DFS (Distributed File System) and supports all major file formats (CSV, Parquet, JSON, Avro, etc.) including popular scientific file formats such as FITS.

Julien Peloton and I worked on this library from scratch, the complete source code can be found [here](https://github.com/JulienPeloton)

## Approach

![Workflow](/assets/img.png)

**Step - 1:** 
Load the data from the source into a raw RDD and then subsequently map to a 3D RDD (Point3DRDD or SphereRDD).

**Step - 2:**
Partition the 3D space (Octree/Onion partitioning) to map points which are in proximity to one another to a uniquely identifiable region. Once such regions are identified, map all data points inside one region to a unique RDD partition. 

**Step - 3:**
Index the RDD to reduce the latency of elements access. (optional)

**Step - 4:**
Apply custom queries (Pixel/Center crossmatch, Range and KNN) on top of the data.

## Topics I worked on - 
1. **JTS replacement for 3D**  
   JTS is a Java library for creating and manipulating 2D vector geometry.

   1. **Main PR**
      1. [https://github.com/astrolabsoftware/spark3D/pull/22](https://github.com/astrolabsoftware/spark3D/pull/22)

   2. **Bug Fix for elements at the boundary**
      1. [https://github.com/astrolabsoftware/spark3D/pull/67](https://github.com/astrolabsoftware/spark3D/pull/67)

2. **Octree** (Custom Spark RDD partitioner)  
   Octree is a 3D extension of Quadtree wherein at each stage cube (instead of square in quadtree case) is split into 8 subcubes. As all the sub node cubes are contained within the parent node and each level contains increasing the level of resolution of details as we move away from the root level, so a search for the data point can be done very easily by moving down the tree along the subcube which contains some information about a particular point. A Leaf node of the Octree is mapped to an unique RDD partition. Object which are in proximity to one another will be mapped to a common RDD partition, thus allowing us to do optimizations and parallelism while processing the 3D data.

   1. Octree base data structure implementation -
      1. [https://github.com/astrolabsoftware/spark3D/pull/30](https://github.com/astrolabsoftware/spark3D/pull/30)
    
   2. Custom Octree partitioner to partition the data across the leaves of the Octree and mapping leaf nodes to a RDD partition -
      1. [https://github.com/astrolabsoftware/spark3D/pull/36](https://github.com/astrolabsoftware/spark3D/pull/36)
   
   3. Avoiding OOM(Out of memory) issue when the data size is very large and collecting even a fraction of the data as a sample causes the driver process to run out of memory - 
      1. [https://github.com/astrolabsoftware/spark3D/pull/56](https://github.com/astrolabsoftware/spark3D/pull/56)
      2. [https://github.com/astrolabsoftware/spark3D/pull/71](https://github.com/astrolabsoftware/spark3D/pull/71)
      
   4. Missing elements after repartitioning fix because of the elements at the boundary of the data limits - 
      1. [https://github.com/astrolabsoftware/spark3D/pull/76](https://github.com/astrolabsoftware/spark3D/pull/76)
    
3. **KNN (K Nearest Neighbours)**  
   Spark based distributed computation implementation of the KNN. A naive approach is to find distance from query point to all other points in data set and pick the top k elements with lowest distance (Note- Spark will parallelize even this computation, thus resulting in lower latency). But with the custom partitioners we know that the nearby objects will lie within the same RDD partition, so initially we can condunct the search just wihtin that partition and then progressively conduct our search in the neighbouring partitions until the limit k is satisfied. 

   1. **KNN Naive** - Iterate through all the elements in the RDD and pick k elements with the lowest distance to the query object.
      1. [https://github.com/astrolabsoftware/spark3D/pull/59](https://github.com/astrolabsoftware/spark3D/pull/59)
    
   2. **KNN efficient** - Perform the search for neighbours first in the neighbourhood of the queryObject which can be identified from the right partition. Then progressively increase in the search space in the neighborhood if the limit is not satisfied.
      1. [https://github.com/astrolabsoftware/spark3D/pull/60](https://github.com/astrolabsoftware/spark3D/pull/60)

   3. **KNN efficient optimizations** - Internal optimization while performing the neighbor search within the partition 
      1. [https://github.com/astrolabsoftware/spark3D/pull/62](https://github.com/astrolabsoftware/spark3D/pull/62)
    
   3. **KNN Unique** - Implementation of KNN which only returns unique elements. This is useful especially when one element can belong to multiple partitions (a case when an object lies within boundary of more than one partition SphereRDD + OctreePartitioning). 
      1. [https://github.com/astrolabsoftware/spark3D/pull/68](https://github.com/astrolabsoftware/spark3D/pull/68)
  
   4. **KNN Ordering bug fix** - 
      Fixed the order in which the KNN were picked within the partitions.
      1. [https://github.com/astrolabsoftware/spark3D/pull/80](https://github.com/astrolabsoftware/spark3D/pull/80)

4. **SphereRDD** (Custom spatial 3D RDD)  
   Once the data is loaded into a rawRDD from the source, each element inside this RDD is mapped to a Sphere object, so that repartitioning based on the custom partitioner and other geometric operation can be applied on top of them. 
   1. **Main PR** - 
      1. [https://github.com/astrolabsoftware/spark3D/pull/38](https://github.com/astrolabsoftware/spark3D/pull/38)

5. **Minor fix** -  
   While partitioning using Onion partitioning, break early when the relevant partition is found, instead of iterating through all the partitions.
   1. [https://github.com/astrolabsoftware/spark3D/pull/27](https://github.com/astrolabsoftware/spark3D/pull/27)
   
## To Do - 
1. Build index to speed-up spatial queries
   
2. RTree partitioning implementation (work in progress)  
   Rtree indexes the objects based on their minimum bounding rectangle. At its leaf level, each of the rectangles will bound a single object. In the next level of the tree, nearby objects would be grouped together and get represented by their own minimum bounding rectangle. While searching, one can start at the topmost level and based on whether query object intersects the bounding rectangles at that level or not, the decision to search within the corresponding subtree will be made.
   [https://github.com/astrolabsoftware/spark3D/pull/87](https://github.com/astrolabsoftware/spark3D/pull/87)

3. spark-fits support on S3, Cassandra

## Relevant links -
1. **spark3D public repo**  
   [https://github.com/astrolabsoftware/spark3D/issues](https://github.com/astrolabsoftware/spark3D/issues)

2. **spark3D website**   
   [https://astrolabsoftware.github.io/](https://astrolabsoftware.github.io/)
