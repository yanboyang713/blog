---
title: "Euclidean Metric"
draft: false
---

## The 3D Euclidean distance formula {#the-3d-euclidean-distance-formula}

\\(d = \sqrt{(x\_2 - x\_1)^2 + (y\_2 - y\_1)^2 + (z\_2 - z\_1)^2}\\)

In this formula, (x1, y1, z1) and (x2, y2, z2) are the coordinates of two points in 3D space, and d is the distance between them. The formula calculates the distance between the two points by taking the square root of the sum of the squares of the differences between their x, y, and z coordinates.


## 3D Euclidean distance between two points {#3d-euclidean-distance-between-two-points}

```c++
#include <iostream>
#include <cmath>

using namespace std;

double euclideanDistance(double x1, double y1, double z1, double x2, double y2, double z2)
{
    double distance = sqrt(pow((x2 - x1), 2) + pow((y2 - y1), 2) + pow((z2 - z1), 2));
    return distance;
}

int main()
{
    double x1, y1, z1, x2, y2, z2;

    cout << "Enter the coordinates of the first point (x1, y1, z1): ";
    cin >> x1 >> y1 >> z1;

    cout << "Enter the coordinates of the second point (x2, y2, z2): ";
    cin >> x2 >> y2 >> z2;

    double distance = euclideanDistance(x1, y1, z1, x2, y2, z2);
    cout << "The 3D Euclidean distance between the two points is: " << distance << endl;

    return 0;
}

```
