## Emission Shader
Emission shader does not cast light onto surfaces in eevee/unreal engine

<figure>
    <img src="images/eevee_emission_material.png" alt="eevee_emission_material" width="200"/>
    <figcaption>Eevee</figcaption>
</figure>
<figure>
    <img src="images/unreal_emission_material.png" alt="unreal_emission_material" width="300"/>
    <figcaption>Unreal Engine</figcaption>
</figure>

Although it does seem to reflect these shaders in shiny surfaces.

If you really want these to cast light you would have to bake to an irradiance field. 

<figure>
    <img src="images/eevee_irradiance_field_setup.png" alt="eevee_irradiance_field_setup" width="300"/>
    <figcaption>Irradiance field setup in Blender</figcaption>
</figure>

<figure>
    <img src="images/irradiance_fields_eevee_result.png" alt="irradiance_fields_eevee_result" width="300"/>
    <figcaption>Irradiance field Eevee result</figcaption>
</figure>

As you can see the results are not very pleasing with low resolution fields.
Increasing resolution increases bake time considerably.  
Unreal Engine might have a similar setup(haven't checked).

