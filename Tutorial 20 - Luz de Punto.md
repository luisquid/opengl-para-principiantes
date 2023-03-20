### Tutorial 20
## Luz de Punto

#### English: https://ogldev.org/www/tutorial20/tutorial20.html

[Click acá](https://youtu.be/ToCSRyXva5w) para ver el video tutorial hecho por ogldev.

## Contexto
We have studied the three basic light models (ambient, diffuse and specular) under the umbrella of directional light. Directional light is a light type which is characterized by a single direction vector and the lack of any origin. Therefore, it doesn't grow weaker with distance (in fact, you can't even define its distance from its target). We are now going to review the point light type which has both an origin as well as a fading effect which grows stronger as objects move away from it. The classic example for a point light is the light blub. You can't feel the fading effect when the light bulb is inside a standard room but take it outside and you will quickly see how limited its strength is. Notice that the direction of light which is constant across the scene for directional light becomes dynamic with point light. That's because a point light shines in all directions equally so the direction must be calculated per object by taking the vector from the object towards the point light origin. That is why we specify the origin rather than the direction for point lights.

The fading effect of point lights is usually called 'attenuation'. The attenuation of a real light is governed by the inverse-square law that says that the strength of light is inversely proportional to the square of the distance from the source of light. This is described in mathematical terms by the following formula:


This formula doesn't provide good looking results in 3D graphics. For example, as the distance becomes smaller the strength of light approaches infinity. In addition, the developer has no control over the results except for setting the initial strength of light. This is too limiting. Therefore, we add a few factors to the formula to make it more flexible:


We've added three light attenuation factors to the denominator. A constant factor, a linear factor and an exponential factor. The physically accurate formula is achieved when setting the constant and linear factors to zero and the exponential factor to 1. You may find it useful to set the constant factor to 1 and the other two factors to a much smaller fraction. When setting the constant factor to one you basically guarantee that the strength of light will reach maximum (actually, what you configure it to be in the program) at distance zero and will decrease as distance grows because the denominator will become greater than one. As you fine tune the linear and exponential factors you will reach the desired effect of light which rapidly or slowly fades with distance.

Let's summarize the steps required for the calculation of point light:

Calculate the ambient term the same as in directional light.
Calculate the light direction as the vector going from the pixel (in world space) to the point light origin. You can now calculate the diffuse and specular terms the same as in directional light but using this light direction.
Calculate the distance from the pixel to the light origin and use it to reach the total attenuation value.
Add the three light terms together and divide them by the attenuation to reach the final point light color.

## El Código Paso a Paso
(lighting_technique.h:24)

struct BaseLight
{
    Vector3f Color;
    float AmbientIntensity;
    float DiffuseIntensity;
};
.
.
.
struct PointLight : public BaseLight
{
    Vector3f Position;

    struct
    {
        float Constant;
        float Linear;
        float Exp;
    } Attenuation;
}
Despite their differences, directional and point lights still have much in common. This common stuff has been moved to the BaseLight structure that both light types are now derived from. The directional light adds the direction in its concrete class while point light adds position (in world space) and the three attenuation factors.

(lighting_technique.h:81)

void SetPointLights(unsigned int NumLights, const PointLight* pLights);
In addition to demonstrating how to implement a point light, this tutorial also shows how to use multiple lights. The assumption is that there will usually be a single directional light (serving as the "sun") and/or possibly several point light sources (light bulbs in a rooms, torches in a dungeon, etc). This function takes an array of PointLight structures and the array size and updates the shader with their values.

(lighting_technique.h:103)

struct {
    GLuint Color;
    GLuint AmbientIntensity;
    GLuint DiffuseIntensity;
    GLuint Position;
    struct
    {
        GLuint Constant;
        GLuint Linear;
        GLuint Exp;
    } Atten;
} m_pointLightsLocation[MAX_POINT_LIGHTS];
In order to support multiple point lights the shader contains an array of structures identical to struct PointLight (only in GLSL). There are basically two methods to update an array of structures in shaders:

You can get the location of each structure field in each of the array elements (e.g. array of 5 structures with 4 fields each leads to 20 uniform locations) and set the value of each field in each element seperately.
You can get the location of the fields only in the first array element and use a GL function that sets an array of variables for each specific field attribute type. For example, if the first field is a float and the second is an integer you can set all the values of the first field by passing an array of floats in one call and set the second field by with an array of integers in the second call.
The first method is more wasteful in terms of the number of uniform locations you must maintain but is more flexible to use. It allows you to update any variable in the entire array by simply accessing its location and does not require you to transform your input data as the second method does.

The second method requires less uniform location management but if you want to update several array elements at once and your user passes an array of structures (as in SetPointLights()) you will need to transform it into a structure of arrays since each uniform location will need to be updated by an array of variables of the same type. When using an array of structures there is a gap in memory between the same field in two consecutive array elements which requires you to gather them into their own array. In this tutorial we will use the first method. You should play with both and decide what works best for you.

MAX_POINT_LIGHTS is a constant value that limits the maximum number of point lights that can be used and must be synchronized with the corresponding value in the shader. The default value is 2. As you increase the number of lights in your application you may end up with a performance problem that becomes worse as the number of lights grows. This problem can be mitigated using a technique called 'deferred shading' which will be explored in the future.

(lighting.fs:46)

vec4 CalcLightInternal(BaseLight Light, vec3 LightDirection, vec3 Normal)
{
    vec4 AmbientColor = vec4(Light.Color, 1.0f) * Light.AmbientIntensity;
    float DiffuseFactor = dot(Normal, -LightDirection);

    vec4 DiffuseColor = vec4(0, 0, 0, 0);
    vec4 SpecularColor = vec4(0, 0, 0, 0);

    if (DiffuseFactor > 0) {
        DiffuseColor = vec4(Light.Color * Light.DiffuseIntensity * DiffuseFactor, 1.0f);
        vec3 VertexToEye = normalize(gEyeWorldPos - WorldPos0);
        vec3 LightReflect = normalize(reflect(LightDirection, Normal));
        float SpecularFactor = dot(VertexToEye, LightReflect);
        if (SpecularFactor > 0) {
            SpecularFactor = pow(SpecularFactor, gSpecularPower);
            SpecularColor = vec4(Light.Color * gMatSpecularIntensity * SpecularFactor, 1.0f);
        }
    }

    return (AmbientColor + DiffuseColor + SpecularColor);
}
It should not come as a big surprise that we can share quite a lot of shader code between directional light and point light. Most of the algorithm is the same. The difference is that we need to factor in the attenuation only for the point light. In addition, the light direction is provided by the application in the case of directional light and must be calculated per pixel for point light.

The function above encapsulates the common stuff between the two light types. The BaseLight structure contains the intensities and the color. The LightDirection is provided seperately because of the reason above. The vertex normal is also provided because we normalize it once when entering the fragment shader and then use it in multiple calls to this function.

(lighting.fs:70)

vec4 CalcDirectionalLight(vec3 Normal)
{
    return CalcLightInternal(gDirectionalLight.Base, gDirectionalLight.Direction, Normal);
}
With the common function in place, the function to calculate the directional light simply becomes its wrapper, taking most of its arguments from the global variables.

(lighting.fs:75)

vec4 CalcPointLight(int Index, vec3 Normal)
{
    vec3 LightDirection = WorldPos0 - gPointLights[Index].Position;
    float Distance = length(LightDirection);
    LightDirection = normalize(LightDirection);

    vec4 Color = CalcLightInternal(gPointLights[Index].Base, LightDirection, Normal);
    float Attenuation = gPointLights[Index].Atten.Constant +
                        gPointLights[Index].Atten.Linear * Distance +
                        gPointLights[Index].Atten.Exp * Distance * Distance;

    return Color / Attenuation;
}
Calculating point light is just a bit more complex than directional light. This function will be called for every configured point light so it takes the light index as a parameter and uses it to index into the global array of point lights. It calculated the vector from the light source (provided in world space by the application) to the world space position passed by the vertex shader. The distance from the point light to the pixel is calculated using the built-in function length(). Once we have the distance we normalize the light direction vector. Remember that CalcLightInternal() expects it to be normalized and in the case of directional light the LightingTechnique class takes care of it. We get the color back from CalcInternalLight() and using the distance that we got earlier we calculate the attenuation. The final point light color is calculated by dividing the color that we have by the attenuation.

(lighting.fs:89)

void main()
{
    vec3 Normal = normalize(Normal0);
    vec4 TotalLight = CalcDirectionalLight(Normal);

    for (int i = 0 ; i < gNumPointLights ; i++) {
        TotalLight += CalcPointLight(i, Normal);
    }

    FragColor = texture2D(gSampler, TexCoord0.xy) * TotalLight;
}
Once we get all the infrastructure in place the fragment shader becomes very simple. It simply normalizes the vertex normal and then accumulates the results of all light types together. The result is multiplied by the sampled color and is used as the final pixel color.

(lighting_technique.cpp:279)

void LightingTechnique::SetPointLights(unsigned int NumLights, const PointLight* pLights)
{
    glUniform1i(m_numPointLightsLocation, NumLights);

    for (unsigned int i = 0 ; i < NumLights ; i++) {
        glUniform3f(m_pointLightsLocation[i].Color, pLights[i].Color.x, pLights[i].Color.y, pLights[i].Color.z);
        glUniform1f(m_pointLightsLocation[i].AmbientIntensity, pLights[i].AmbientIntensity);
        glUniform1f(m_pointLightsLocation[i].DiffuseIntensity, pLights[i].DiffuseIntensity);
        glUniform3f(m_pointLightsLocation[i].Position, pLights[i].Position.x, pLights[i].Position.y, pLights[i].Position.z);
        glUniform1f(m_pointLightsLocation[i].Atten.Constant, pLights[i].Attenuation.Constant);
        glUniform1f(m_pointLightsLocation[i].Atten.Linear, pLights[i].Attenuation.Linear);
        glUniform1f(m_pointLightsLocation[i].Atten.Exp, pLights[i].Attenuation.Exp);
    }
}
This function updates the shader with the point lights values by iterating over the array elements and passing each element's attribute values one by one. This is the so called "method 1" that was described earlier.

This tutorials demo shows two point lights chasing one another across a field. One light is based on the cosine function while the other on the sine function. The field is a very simple quad made of two triangles. The normal is a straight up vector.