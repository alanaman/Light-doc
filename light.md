## Shadow mapping

In this method of calcuting shadows, we render a depth map from the lights point of view. The surfaces that the light falls on will be rendered.

For directional lights it is rendered onto a 2D texture. For point and spot lights it is rendered onto a cubemap. For spot lights we could get away with a 2D texture but we'll have to limit the frustrum(could be made adaptive for performance).

To start we query all the light sources and make a call for rendering depthmaps for each of these lights.

``` cpp
std::vector<PointLight> pointLights;
std::vector<SpotLight> spotLights;
std::vector<DirectionalLight> directionalLights;

//query and fill the vectors

for (unsigned int i = 0; i < directionalLights.size(); i++)
    renderShadows(scene, directionalLights[i], i);
for (unsigned int i = 0; i < pointLights.size(); i++)
    renderShadows(scene, pointLights[i], i);
for (unsigned int i = 0; i < spotLights.size(); i++)
    renderShadows(scene, spotLights[i], i);
```

```renderShadows()``` makes calls to the renderer to render each object onto a depthmap. These depthmaps were created in the constructor for the ```SceneRenderer```. Since they are pre-created their needed to be a cap on their number(currently 4 for each type for light).

GLSL code for rendering depth onto cubemaps:

Vertex shader
``` glsl
#version 330 core
layout(location = 0) in vec3 a_Position;
uniform mat4 u_transform;
void main()
{
	gl_Position = u_transform * vec4(a_Position, 1.0);
}
```
Geometry shader: Sends out 6 triangles in different layers, as seen from 6 faces of the cube, for each input triangle.
``` glsl
#version 330 core
layout (triangles) in;
layout (triangle_strip, max_vertices=18) out;
uniform mat4 shadowMatrices[6]; // projection matrix for each face of the cube
out vec4 FragPos; // FragPos from GS (output per emitvertex)
void main()
{
    for(int face = 0; face < 6; ++face)
    {
        gl_Layer = face; // built-in variable that specifies to which face we render.
        for(int i = 0; i < 3; ++i) // for each triangle's vertices
        {
            FragPos = gl_in[i].gl_Position;
            gl_Position = shadowMatrices[face] * FragPos;
            EmitVertex();
        }    
        EndPrimitive();
    }
}
```
Fragment shader
``` glsl
#version 330 core
in vec4 FragPos;
uniform vec3 lightPos;
uniform float far_plane;
void main()
{
    float lightDistance = length(FragPos.xyz - lightPos);
    
    // map to [0;1] range by dividing by far_plane
    lightDistance = lightDistance / far_plane;
    
    // write this as modified depth
    gl_FragDepth = lightDistance;
}
```

The rendered depthmaps are then bound to Texture Units 4 through 15 (3*4=12 units used).
Some kind of texture management system needs to be implemented to use Texture Units more efficiently.

Now the main shader is used to render out the actual scene.
In this shader the following functions are used to determine if a pixel is in shadow or not.

For point/spot lights:
``` glsl
float calculateShadow(PointLight light)
{
    // get vector between fragment position and light position
    vec3 fragToLight = v_worldPos - light.position;
    // use the light to fragment vector to sample from the depth map    
    float closestDepth = texture(light.depthCubemap, fragToLight).r;
    // it is currently in linear range between [0,1]. Re-transform back to original value
    closestDepth *= light.far_plane;
    // now get current linear depth as the length between the fragment and light position
    float currentDepth = length(fragToLight);
    // now test for shadows
    float bias = 0.05; 
    float shadow = currentDepth -  bias > closestDepth ? 1.0 : 0.0;

    return shadow;
}  
```
For directional lights:
``` glsl
float calculateShadow(DirectionalLight light)
{
	vec4 fragPosLightSpace = light.lightSpaceMatrix * vec4(v_worldPos, 1.0);
    vec3 projCoords = fragPosLightSpace.xyz / fragPosLightSpace.w;

    projCoords = projCoords * 0.5 + 0.5;
    float closestDepth = texture(light.depthMap, projCoords.xy).r; 

    float currentDepth = projCoords.z;
    float bias = 0.005;
	float shadow = (currentDepth - bias > closestDepth ) ? 1.0 : 0.0; 
    if(projCoords.z > 1.0)
        shadow = 0.0;
        
    return shadow;
}
```