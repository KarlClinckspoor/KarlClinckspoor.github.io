---
layout: post
title: Positioning stuff in ManimCE
description: The jury rigged solution I used to make positioning stuff easier
author: Karl Jan Clinckspoor
date: 11 April 2021
tags:
- python
- manim
---

I'm exploring manimCE and I've been having some trouble placing things and aligning them. 
Specifically, I wanted to create a `GraphScene` and place the axis labels at specific spots, not 
where they're placed automatically. The way I discovered at the moment (which is probably a bit 
boneheaded) is to hard code their positions. For example:

```python
class RheologyPlot5(GraphScene):
    def __init__(self, **kwargs):
        GraphScene.__init__(
            self,
            x_min=-1,
            x_max=10,
            num_graph_anchor_points=100,
            ymin=-1,
            ymax=10,
            graph_origin=3 * DOWN + 4.5 * LEFT,
            axes_color=WHITE,
            x_labeled_nums=range(0, 11, 1),
            y_labeled_nums=range(0, 11, 1),
            x_axis_label=r"",
            y_axis_label=r"",
        )
        x_axis_label = Tex(r"$\dot{\gamma}/s^{-1}$").move_to([4.5, -3, 0])
        y_axis_label = (Tex(r"$\tau/m\text{Pa} \cdot s$").move_to([-5.5, 0, 0]).rotate(90 * DEGREES))
        self.add(x_axis_label, y_axis_label)
```

I'm placing the labels at `(X, Y, Z)` positions `[4.5, -3, 0]` and `[-5.5, 0, 0]`. To figure 
these numbers, instead of fiddling with them and compiling the scene, I created a custom class 
that overlays a grid over the whole scene (hopefully) and makes aligning easier. I have the 
nagging feeling that there's a better way to do this, but I'm new to manim, and this was a good 
exercise.

This is the class I created. Every 1 unit, it traces another horizontal or vertical line, and 
every other line, it's colored red. Then, each square has 0.2 subdivisions, so more precise 
alignment is easier.

```python
class TestCoordinates(Scene):
    def add_grid(self):
        minimum = -8
        maximum = 8
        horizontal_lines = []
        vertical_lines = []
        colors = [RED, WHITE]
        for i in range(0, maximum - minimum + 1):
            line_v = Line(
                start=[minimum + i, minimum, 0],
                end=[minimum + i, maximum, 0],
                color=colors[i % 2],
            ).set_opacity(0.5)
            line_h = Line(
                start=[minimum, minimum + i, 0],
                end=[maximum, minimum + i, 0],
                color=colors[i % 2],
            ).set_opacity(0.5)
            vertical_lines.append(line_v)
            horizontal_lines.append(line_h)
            for j in range(4):  
                # Dashed lines are too slow, changed to normal lines
                # line_v = DashedLine(start=[minimum + i + (j+1) * 0.2, minimum, 0], end=[minimum + i + (j+1) * 0.2, maximum, 0]).set_opacity(0.5)
                # line_h = DashedLine(start=[minimum, minimum + i + (j+1) * 0.2, 0], end=[maximum, minimum + i + (j+1) * 0.2, 0]).set_opacity(0.5)
                line_v = Line(
                    start=[minimum + i + (j + 1) * 0.2, minimum, 0],
                    end=[minimum + i + (j + 1) * 0.2, maximum, 0],
                ).set_opacity(0.2)
                line_h = Line(
                    start=[minimum, minimum + i + (j + 1) * 0.2, 0],
                    end=[maximum, minimum + i + (j + 1) * 0.2, 0],
                ).set_opacity(0.2)
                vertical_lines.append(line_v)
                horizontal_lines.append(line_h)
                vertical_lines.append(line_v)
                horizontal_lines.append(line_h)

        centerDot = Dot(ORIGIN, 0.1, color=RED)

        self.add(*horizontal_lines, *vertical_lines)
        self.add(centerDot)
```

Finally, if you want to use this, you can inherit from two classes and use the `add_grid` function.

```python
class RheologyPlot5(GraphScene, TestCoordinates):
    ...
    def create(self):
        ...
        self.add_grid()
        ...
```

And there you have it, a grid to position text on your scene.
