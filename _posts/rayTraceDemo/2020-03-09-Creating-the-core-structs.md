---
layout: post
title: "01 - Creating the core structs and classes"
description: "In this series of blog posts I want to try to write a RayTracer from scratch using plain C#"
img: "rayTracer/Recursive_raytrace_of_a_sphere.jpg"
fig-caption: "Image source: https://de.wikipedia.org/wiki/Raytracing"
permalink: /raytracer/01/
tags: [c#, ray tracing]
---

---

> GitHub repository : [RayTracingDemo on GitHub](https://github.com/Cxyda/RayTracingDemo)
>
> Chapter project files : [Chapter 01](https://github.com/Cxyda/RayTracingDemo/commit/ba8242f9adb523fa11356774da350c726eee9dbb)

---


## Chapter 01

In this chapter I will start to create all the core structs and classes we need for the ray tracer. For now I see structs like `Vector`, `Sphere`, `Color`, `Ray` and `Hit` as well as classes like `Scene`, `Camera` and `Light`. In this chapter we will implement only the first 3 which are `Vector`, `Sphere`, and `Color`.

Let's start with the `Vector` struct.
 
#### The vector struct:

As I mentioned in the introduction, I want to write a ray tracer from scratch. This includes writing a very basic model like the `Vector` which is capable for storing `X, Y, Z` and `W` values for the corresponding position.



{% highlight csharp %}

public struct Vector
{
    public static Vector Zero = new Vector();
    
    private double _magnitude;
    private double _sqrMagnitude;
    
    public readonly double X, Y, Z, W;
    
    public Vector(double x = 0.0, double y = 0.0, double z = 0.0, double w = 0.0)
    {
        X = x;
        Y = y;
        Z = z;
        W = w;

        _sqrMagnitude = double.NaN;
        _magnitude = double.NaN;
    }
}

{% endhighlight %}

We start simple and add variables for the X,Y,Z and W values. The reason why we also store the W value is because we will need it later for the matrices. We could of course create two different vector structs (Vector3 and Vector3) for example but for now I'm too lazy to write all the boiler-plate code.

We also add variables for the magnitude and the square magnitude. The magnitude is the real length of the vector, while the square magnitude is the, guess what, the squared magnitude value. This is a small optimization. In some cases we are not interested in the real length of a vector, but only in the length ratio between different vectors for example when we want to compare them. In this case we can save us the square root computation of the magnitude.

{% highlight csharp %}
public struct Vector
{
    //..
    public double Magnitude
    {
        get
        {
            if (double.IsNaN(_magnitude))
                _magnitude = Math.Sqrt(SqrMagnitude);
            return _magnitude;
        }
    }
    public double SqrMagnitude
    {
        get
        {
            if (double.IsNaN(_sqrMagnitude))
                _sqrMagnitude = X * X + Y * Y + Z * Z + W * W;
            return _sqrMagnitude;
        }
    }
    //..
}
{% endhighlight %}

We also need getters to access the magnitude values. For this I use a strategy called *[memoization](https://en.wikipedia.org/wiki/Memoization)*. 
This fancy word only means that we cache the result of an calculation. This is mostly used for operations which are very 
expensive to calculate. In our case that's not really the case but since we not always require the magnitude of a vector 
and the values of a vector never change, we only calculate the magnitude when it is needed ad store it afterwards.

Now we need to implement all necessary math operations like `+, -, *, == and !=` for our vectors. We don't need to override the `/` operator because we can multiply with the inverse value instead.

{% highlight csharp %}
public struct Vector
{
    //..
    public static Vector operator +(Vector a, Vector b)
    {
        return new Vector(a.X + b.X, a.Y + b.Y, a.Z + b.Z, a.W + b.W);
    }
    public static Vector operator -(Vector a, Vector b)
    {
        return new Vector(a.X - b.X, a.Y - b.Y, a.Z - b.Z, a.W - b.W);
    }
    public static Vector operator *(Vector a, double scalar)
    {
        return new Vector(a.X * scalar, a.Y * scalar, a.Z * scalar, a.W * scalar);
    }
    public static Vector operator *(double scalar, Vector a)
    {
        return a * scalar;
    }
    public static bool operator ==(Vector a, Vector b)
    {
        return Math.Abs(a.X - b.X) < double.Epsilon &&
               Math.Abs(a.Y - b.Y) < double.Epsilon &&
               Math.Abs(a.Z - b.Z) < double.Epsilon &&
               Math.Abs(a.W - b.W) < double.Epsilon;
    }
    public static bool operator !=(Vector a, Vector b)
    {
        return !(a == b);
    }
}
{% endhighlight %}

... and of course we weed to be able to `Normalize` and `Reflect` the vector as well as calculate the `Dot`-product and the `Cross`-product.

##### Cross and dot product:

{% highlight csharp %}
public struct Vector
{
    //..
    public void Normalize()
    {
        this = Normalize(this);
    }

    public static Vector Normalize(Vector a)
    {
        return new Vector(
            a.X / a.Magnitude, 
            a.Y / a.Magnitude, 
            a.Z / a.Magnitude, 
            a.W / a.Magnitude);
    }
    public static Vector Cross(Vector a, Vector b)
    {
        return new Vector(
            a.Y * b.Z - a.Z * b.Y,
            a.Z * b.W - a.W * b.Z,
            a.W * b.X - a.X * b.W, 
            a.X * b.Y - a.Y * b.X );
    }

    public static double Dot(Vector a, Vector b)
    {
        return a.X * b.X + a.Y * b.Y + a.Z * b.Z + a.W * b.W;
    }

    /// <summary>
    /// The incoming vector needs to be normalized!
    /// </summary>
    public static Vector Reflect(Vector incoming, Vector normal)
    {
        return incoming - 2.0 * Dot(incoming, normal) * normal;
    }
}
{% endhighlight %}


As you probably know, the **[cross product](https://en.wikipedia.org/wiki/Cross_product)** is used to calculate the vector which is perpendicular to two other vectors. 
We will need this later for our ray tracer.

The **[dot product](https://en.wikipedia.org/wiki/Dot_product)** returns the relationship of two vectors, namely the cosine of the angle between them. But we don't have to calculate the angle to get knowledge out of the dot product. There are actually three cases the dot-product can return:

    1) a * b > 0    => the angle between the vector a and b is less than 90 degrees
    2) a * b = 0    => the angle between vector a and b is equal to 90 degrees
    3) a * b < 0    => the angle between vector a and b is greater than 90 degrees

    (*where a and b are vectors; a * b is read as "the dot product of vectors a and b")

<br>

<figure>
<img src="{{site.baseurl}}/assets/img/rayTracer/dot-product.png"
     alt="The dot product cases"
     style="margin-left: auto; margin-right: auto; display: block; width:50%" />
<figcaption>Fig.1 - The different dot product cases.</figcaption>
</figure>

---

#### The Color struct:

The `Color` struct represents the basic functionality we need to implement different colors in our ray tracer. A color 
will be represented by 4 float values (R, G, B, A) in the range of [0.0f, 1.0f], if we allow HDR colors we would not clamp 
the color at 1.0f but let the value exceed this limit. Those values represent of course the 4 different channels red (R), 
green (G), blue (B) and alpha (A)

{% highlight csharp %}
public struct Color
{
    public readonly float R;
    public readonly float G;
    public readonly float B;
    public readonly float A;
    public readonly bool HdrColor;

    public Color(float r, float g, float b, float a = 1f, bool hdrColor = false)
    {
        R = r;
        G = g;
        B = b;
        A = a;
        HdrColor = hdrColor;
        
        A = Clamp(A);
        if (HdrColor) return;

        R = Clamp(R);
        G = Clamp(G);
        B = Clamp(B);
    }

    private float Clamp(float value)
    {
        if (value < 0)
            value = 0f;

        else if (value > 1f)
            value = 1f;

        return value;
    }
}
{% endhighlight %}

We also need functionality to add colors together and multiply them. In both cases we just add / multiply each channel 
value with the corresponding channel value of the other color. This means that when we add colors together, the resulting 
color will be brighter and when we multiply, it will be darker (as long as we don't use HDR colors).

{% highlight csharp %}
public struct Vector
{
    //..
    public static Color operator +(Color a, Color b)
    {
        return new Color(a.R + b.R, 
                         a.G + b.G, 
                         a.B + b.B, 
                         a.A + b.A, a.HdrColor || b.HdrColor);
    }
    public static Color operator *(Color a, double intensity)
    {
        return a * (float) intensity;
    }
    public static Color operator *(Color a, float intensity)
    {
        return new Color(a.R * intensity,
                         a.G * intensity, 
                         a.B * intensity, 
                         a.A * intensity, a.HdrColor);
    }
    public static Color operator *(Color a, Color b)
    {
        return new Color(a.R * b.R, 
                         a.G * b.G, 
                         a.B * b.B, 
                         a.A * b.A, a.HdrColor || b.HdrColor);
    }
}
{% endhighlight %}

Ok! Now we implemented `Vector`s and `Color`s in our ray tracer, but we need some more data structures before we finally 
can see something. For example some geometric objects!

---

#### The Sphere struct

The first shape we implement will be the sphere. This is the simplest object when it comes to calculation of it's surface and intersections. 
I is indeed very simple and might be simpler then you might expect.

{% highlight csharp %}
public struct Sphere
{
    public int Id { get; }

    public readonly Vector Center;
    public readonly double Radius;
    
    public Sphere(int id, Vector center, double radius)
    {
        Id = id;
        Center = center;
        Radius = radius;
    }
}
{% endhighlight %}

That's it! (for now). Super simple. We just need a vector which specifies its position and a radius value. The id is a unique identifier we will need later. 
Now we can represent spheres in our scenes. Wait! Scenes?


That's it for this chapter. We did some ground work for our ray tracer. Next chapter we will implement the remaining structs and classes and see the first ray traced results!


Read: [previous](/blog/raytracer/00/) | next