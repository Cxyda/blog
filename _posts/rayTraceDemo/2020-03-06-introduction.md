---
layout: post
title: "00 - Writing a RayTracer from scratch (Introduction)"
description: "In this series of blog posts I want to try to write a RayTracer from scratch using plain C#"
img: "rayTracer/Recursive_raytrace_of_a_sphere.jpg"
fig-caption: "Image source: https://de.wikipedia.org/wiki/Raytracing"
permalink: /raytracer/00/
tags: [c#, ray tracing]
---

---

> GitHub repository : [RayTracingDemo on GitHub](https://github.com/Cxyda/RayTracingDemo)

---

Welcome to this new series where I want to write a small RayTracer from scratch. 

#### Motivation:
You are probably asking why I want to write it from scratch. Well, that's a valid question and there are mainly two reasons for it:

- First I wanted to do a project outside of the Unity3D eco system I'm working in during my professional work day.
- Second I want to prove to my self that I have understood the concept of ray tracing and the math behind it, without relying on 'magic' which comes out of frameworks and libraries.

#### Project goals:

- Diffuse ray tracing (for now)
- Having a scene with **simple geometric objects** (spheres, cubes, planes,..)
- The objects have a material with **diffuse color, specular and glossiness** (no transparent/translucent materials for now)      
- Having (multiple) light sources (point light, area light)
    - lights are casting shadows


Since I'm developing this project on *Windows* and *MacOS* and I wasn't able to find a simple multi platform canvas framework on my quick google search I decided to go even more low level for now and dump the result of my ray tracer into a **single image** file. The file-format The most simple file format I found for this was the **.ppm** file-format.

For now the project is limited to roughly two weeks are my rare spare time. Therefore I'll keep it simple for now and implement the simplest ray tracing technique, the diffuse ray tracing. This technique allows soft shadows but no global illumination. Maybe I will extend it a later and implement path tracing.

Read: previous | [next](/blog/raytracer/01/)

