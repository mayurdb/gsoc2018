## Spark3D

Spark3D is as an open-source Spark library to process large scala 3D data-sets from, Astrophysics, High Energy Physics, Meteorology, etc. The library works on top of the latest Spark versions (2.0+), has user friendly APIs (in Scala and Python (ongoing)), works on top of HDFS and supports all major file formats (CSV, Parquet, JSON, Avro, etc.) including popular scientific file formats such as FITS.

## Approach

![Workflow](/assets/img.png)

Step - 1: 
Load the data from the source into a raw RDD and then subsquently mapped to a 3D RDD (Point3DRDD or SphereRDD).

Step - 2:
Partition the 3D space (Octree/Onion partitioning) to map points which are in proximity to one another to a uniquely identifiable region.

Step - 3:
Repartition the RDD with custom Octree or Onion partitioner (based on which partitioning scheme was used in Step-2) to store data points inside the region identified in Step-2 in a unique partition. 

Step - 4:
Index the RDD to reduce the latency of elements access. (optional)

Step - 5:
Apply custom queries (Pixel/Center cross match, Range and KNN) on top of the data.

## Topics worked on - 
1. JTS (a Java library for creating and manipulating 2D vector geometry.) replacement for 3D
   A) Main PR 
      https://github.com/astrolabsoftware/spark3D/pull/22

   B) Bug Fix
      https://github.com/astrolabsoftware/spark3D/pull/67

2. Octree
   A) Octree implementation
      https://github.com/astrolabsoftware/spark3D/pull/30
    
   B) Octree partitioner
      https://github.com/astrolabsoftware/spark3D/pull/36
   
   C) Bug fixes
     a) Sample size related
        https://github.com/astrolabsoftware/spark3D/pull/56
        https://github.com/astrolabsoftware/spark3D/pull/71
     b) Missing elements after repartitioning
        https://github.com/astrolabsoftware/spark3D/pull/76
    
3. KNN
   A) KNN Naive
      https://github.com/astrolabsoftware/spark3D/pull/59
    
   B) KNN efficient
      https://github.com/astrolabsoftware/spark3D/pull/60
      https://github.com/astrolabsoftware/spark3D/pull/62
    
   C) KNN Unique
      https://github.com/astrolabsoftware/spark3D/pull/68/files
  
   D) KNN Ordering bug fix
      https://github.com/astrolabsoftware/spark3D/pull/80

4. SphereRDD
   A) Main PR
      https://github.com/astrolabsoftware/spark3D/pull/38

5. Other minor fix
   https://github.com/astrolabsoftware/spark3D/pull/27
   

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/mayurdb/mayurdb.github.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
