### Shaders, Continued

Asset creation is not terribly exciting to talk about! But we've done this spiel a dozen times by now so assume that I continue modelling and texturing and shading and writing dialogue.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/tuxremodel.png "This is the LAST time I remodel this ship. I swear to god.")

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/shiplayout.png "Big suckers.")

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/fuelmodule.png "Looks kinda like a bottle.")

In addition to this, the actual art assets are getting an upgrade!

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/stevenupgrade.png "Thank god.")

Much thanks to the artist in question.

There's also a whole chunk of dialogue that has been written (I think pretty much every wingman has at least 4 solo conversations now, which is a shit-load of writing if you stop to think about it for a second given that there are eight of them), but that's kind of hard to show off too. It's good, though. At least, *I* think so, and since this whole damn project is just me throwing a hissy fit about space sims my opinion is the one that matters.

## So Why The Post?

I'm getting my shit together with shaders recently and I figured it'd be useful to continue posting about them. [Last time I discussed an extremely basic shader and the graphics pipeline](https://wizard-of-chaos.github.io/2023/02/27/shaders.html).

> Your GPU only knows a few things and it only knows how to do a few things. It's good with basic math, it's good with general parallel problems, and it's good with processing all of these as quickly as it possibly can. There is a whole *wealth* of things that it does *not* know, though, which you need to *tell* it. Things like where your model is in the world, for instance, or what color it is, or what the texture looks like -- hell, it doesn't even remember what it *just did*.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/pipeline.png "Rendered lovingly with a drawing tablet and about two minutes.")

I was getting pretty fed up trying to simultaneously figure out shaders *and* how to incorporate them into Irrlicht *and* how they play into the game and it's a bitch and a half to try and identify what's going wrong with my shader *while* I'm getting shot at. To that end, I wrote a material renderer so I could just get a handle on this crap.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/renderer.gif "Yay!")

You, too, can mess around with HLSL shaders, since the code is [up on my Github right here](https://github.com/Wizard-Of-Chaos/ShaderRenderer). In any case, armed with the tool I set out to go write a normal-map shader that uses it as a bump map and see what I could do with regards to a normal shader.

## Normal Maps and What They Are

[Wikipedia describes a normal map](https://en.wikipedia.org/wiki/Normal_mapping) as "a texture mapping technique used for faking the lighting of bumps and dents – an implementation of bump mapping". That's correct. We know it is correct because it is on Wikipedia.

Bump maps are just one of many ways graphical designers try to trick you into seeing more detail than is actually there. The simplest version of a bump-map is probably just a height map, which would be a black and white representation of the *height* of the given texture. I'm a lazy sack of crap, and so are most artists, and so are most programmers, for that matter. Nobody wants to spend a bunch of time modelling, for example, the veins on someone's arm, or the actual grain on a metal texture and baking these things straight into the model. Instead, what the consortium of lazy people do is toss in some representation of the arm that has the *height* where the veins should go, and that gets rendered as lighter or darker depending on how "high" it is compared to the rest of the arm. If you've ever seen a USGS topographical map, it's basically the same concept.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/usgs_map.png "Look, bumps!")

Normal maps take this basic idea of "faking" height and texture and go a bit further with it. Each pixel in a normal map corresponds to the *surface normal* on the texture at the corresponding pixel -- which just means the direction perpendicular to the surface, or to simplify it even further, the local "up" for that spot on the texture. Normal maps tend to look somewhat psychedelic because the information for the surface normal -- the X, Y, and Z corresponding to the vector defining "up" -- is encoded as the RGB values on the actual image itself. If you're a complete freak, you can even hand-paint these things. I never got the hang of it.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/handpainted_norm.png "Did someone let the 60s in here?")

[As outlined in that article](https://magazine.artstation.com/2019/04/handpainting-normal-maps-in-photoshop-with-nick-lewis/), you apply the normal map to the previous texture and end up with a lot more depth and, well, *texture* on the finished product.

So what does this mean for us, the hack game developers trying to write shaders for these things? Well...

## Extra Inputs

The last time I talked about shaders, we wound up with a shader that looks like this.

```hlsl
float4x4 world;
float4x4 view;
float4x4 projection;

float4 ambientColor = float4(.4, .5, 1.0, 1.0);
float ambientIntensity = .5;

struct VERTEX_INPUT
{
    float4 position : POSITION0;
}

struct VERTEX_OUTPUT
{
    float4 position : POSITION0;
}

VERTEX_OUTPUT myVertexShader(VERTEX_INPUT input)
{
    VERTEX_OUTPUT out;
    float4 worldPos = mul(input.position, world);
    float4 viewPos = mul(worldPos, view);
    output.position = mul(viewPos, projection);
    return out;
}

struct PIXEL_OUTPUT
{
    float4 color : COLOR0;
}

PIXEL_OUTPUT myPixelShader(VERTEX_OUTPUT in)
{
    PIXEL_OUTPUT out;
    out.color = ambientColor * ambientIntensity;
    return out;
}
```

This is, again, an incredibly basic shader that simply colors in the *entire mesh* with the ambient color and the ambient intensity. It doesn't do anything with textures, it doesn't do anything with light sources, and it doesn't do anything with normal maps.

The information we're going to need to write a normal-map version of this is way more substantial. We need to know all the previous stuff, and then on top of that we need to know things about the *light source* -- where it is relative to the model, what color it is, where the *camera* looking at this object is, and we're going to need access to both the starting texture and the normal map to blend these things together. We're also going to need to know the vector *tangential* to the surface -- going along the surface -- and the *binormal* vector, which is perpendicular to the tangent vector. This, combined with the outward-facing surface normal, gives us all the information we need about how light is going to hit this thing.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/normtangbinorm.png "Thank you, mspaint.")

If you want to make every graphic designer / modeller / programmer / mathematician in an eight-mile radius spit blood, you can think of the normal, tangent, and binormal as up, forward, and right. Having all three directions allows you to define orientation. So, to recap, we're going to need some additional orientation information for the texture, the camera position, and light information.

Got it? Let's get started!

## Design

Our constants are going to look like this.

```hlsl
float4x4 WORLD; //world transform
float4x4 VIEW; //view transform
float4x4 PROJ; //projection transform
float4x4 INV_TRANSPOSE; //world inverse, transposed
float3 LIGHT_POS; //light position
float4 LIGHT_DIFFUSE_COLOR; //light color
float4 LIGHT_AMBIENT_COLOR; //ambient light color
float4 LIGHT_SPECULAR_COLOR; //light specular color
float3 CAMERA_VIEW; //camera view vector
```

We're going to start off with the world, view, and projection matrices from last time that allow us to stuff the model into world space. We're also going to neeed the transposed inverse world matrix -- this is going to be used to transform the normal vectors we have into object space so we can do math with it. We also need the position of the light, a couple different colors from the light (ambient for general work, diffuse for light sources, and specular for "glare"), and the aforementioned camera view. For simplicity's sake, I'm not going to mess around with any intensity factors.

Our input this time looks like this:

```hlsl
struct VertexInput
{
    float4 Position : POSITION;
    float4 Normal : NORMAL0;
    float2 TextureCoordinate : TEXCOORD0;
    float3 Tangent : TEXCOORD1; //IMPORTANT: Irrlicht passes in Tangent and Binormal information as TEXCOORD1 and TEXCOORD2. Wild.
    float3 Binormal : TEXCOORD2;
};
```

If you're not using the irrlicht engine, the Tangent and Binormal info will be in TANGENT0 and BINORMAL0 respectively. I have no idea why my engine does this. Do note the texture coordinate, though, which is a new variable we're adding to say to the shader just where the hell we are on the actual texture iamge.

The vertex output will now look like:

```hlsl
struct VertexOutput
{
    float4 Position : POSITION;
    float2 TextureCoordinate : TEXCOORD0;
    float3 Normal : TEXCOORD1;
    float3 Tangent : TEXCOORD2;
    float3 Binormal : TEXCOORD3;
    float3 LightDirection : TEXCOORD4;
    float LightIntensity : TEXCOORD5;
};
```

Because we're working from the light position, we're not going to have access to the *direction* of the light (although we would if it were a spotlight instead of a point light -- that's a different implementation style). If you want to rip the direction of the light to the model in... whatever you're doing, feel free to do that and disregard the last two parameters. The normal, tangent, and binormal are what we're mainly after.

The vertex main function will be similar to last time, where we stuff the vertex coordinates into world space. This time, though, we're doing a few extra calculations.

```hlsl
VertexOutput vertexMain(VertexInput input)
{
    VertexOutput ret;
    ret.Position = mul(mul(mul(input.Position, WORLD), VIEW), PROJ);
    ret.Normal = normalize(mul(input.Normal, INV_TRANSPOSE));
    ret.Tangent = normalize(mul(input.Tangent, INV_TRANSPOSE));
    ret.Binormal = normalize(mul(input.Binormal, INV_TRANSPOSE));

    float3 direction = LIGHT_POS - input.Position;
    direction = normalize(direction);
    
    float intensity = dot(ret.Normal, direction);
    ret.LightIntensity = intensity * 10;
    ret.LightDirection = direction;

    ret.TextureCoordinate = input.TextureCoordinate;

    return ret;
}
```

Just like last time we're going to calculate where the model is in "world" space, and throw that into position, but we're also going to transform the normal vectors we have (normal, binormal, and tangent) by the inverse transpose world, so those will end up in model space as well. In addition, we're also calculating the direction of the light source to the model and how intense it is based on the normal -- remember this is the "up" vector -- and how that corresponds to the direction. If the "up" is aligned with the light direction, that means it's on the other side of the model and not being lit, for example. Intensity there is used to define just how much light should be getting to this vertex. I multiplied it by 10 to make it more obvious.

Now for the fun stuff. First we're going to need to grab the textures -- doing that with a texture sampler that does exactly what it says on the tin.

```hlsl
sampler2D textureSampler : register(s0);
sampler2D bumpSampler : register(s1);
```

And then we'll use those and the vertex output on our pixel shader.

```hlsl
loat4 pixelMain(VertexOutput input) : COLOR0
{
    //.15 is the bump constant
    float3 bump = .15 * (tex2D(bumpSampler, input.TextureCoordinate) - (.5, .5, .5));
    float3 bumpNormal = input.Normal + (bump.x * input.Tangent + bump.y * input.Binormal);
    bumpNormal = normalize(bumpNormal);
    
    float intensity = dot(normalize(input.LightDirection), bumpNormal);
    if (intensity < 0)
        intensity = 0;
    
    float3 light = normalize(input.LightDirection);
    float3 r = normalize(2 * dot(light, bumpNormal) * bumpNormal - light);
    float3 v = normalize(mul(normalize(CAMERA_VIEW), WORLD));
    float dotRV = dot(r, v);
    
    //5 here is how shiny it is and the 1 is how intense the spec highlight is
    float4 spec = 1 * LIGHT_SPECULAR_COLOR * max(pow(dotRV, 5), 0) * intensity;
    
    float4 texColor = tex2D(textureSampler, input.TextureCoordinate);
    texColor.a = 1;
    
    return saturate(texColor * (intensity * LIGHT_DIFFUSE_COLOR) + spec + LIGHT_AMBIENT_COLOR);

}
```

There are a few magic numbers in here that I should explain. The .15 when calculating the "bump" there is determining just how exaggerated the effect of the normal map should be. This can be passed in as a constant if you like. It gets the normal information from the texture coordinate, then multiplies it all together by the prior vertex normal/binormal/tangent information to determine what the final normal should be, and from that we can figure out how intense the light needs to be on tihs particular spot on the texture.

Additionally, the values "r" and "v" correspond to the *reflection* vector and the *view* vector, the first determining just how light will be reflecting once it bounces off whatever we're looking at and then the *view* vector which tells us how we're *looking* at the object. THose two are used for specular highlights, which you've probably seen in real life as "glare". The specular highlight color is then calculated using the dot product of the reflection and the view vector, with the magic numbers 1 and 5 determining how intense and shiny the surface is respectively (again, these can be passed in as material constants).

Once we have all that we can get the color of the texture, multiply it by the intensity of the light as determined by the normal map, add any ambient color and the specular highlight (if applicable) and... that's it!

## What's Next?

Trying to get the game into an actual, honest-to-god releaseable state. That means a lot of remodels and retexturing and shaders and whatnot. Plenty of work. But I can see the light at the end of the tunnel! It's lookin' great.