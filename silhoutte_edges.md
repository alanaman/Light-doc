### Creating an efficient algorithm for calculating Silhoutte edges

#### Targets
* Transform the vertex positions to clip/world space using openGL(GPU)
  * Requires reading output from vertex/geometry shader.(checking out Transform Feedback)

* Should ideally work with all possible meshes

    #### Problems
    1. Unclosed geometry
    2. non-manifold edges
    3. smoothed/bad normals
    #### Possible Solution
    If we recalulate normals for each face, only problem 3 will be solved.
    So lets try a different approach.  
    First we convert all the vertex positions to clip space using vertex/geo shader. 
    Looking through the camera each edge will be a straight line segment.  
    Then we consider all the faces attached to a particular edge. 
    Each of these faces will have a vertex that does not make up the edge under consideration.
    If all of these vertices lie on the same side of the edge then that edge is sihoutte edge.
    Else it is not.

