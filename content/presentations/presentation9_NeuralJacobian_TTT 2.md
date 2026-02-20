---
title: "Analyzing Neural Jacobian Methods in Applications of Visual Servoing and Kinematic Control" 
date: 2021-07-06
tags: ["Learning from demonstration", "Teleoperation", ]
author: ["Michael Przystupa"]
description: "Tea Time Talk on Neural Jacobian Methods"
summary: "Research Presentation on my work analyzing Neural Jacobian Methods"
editPost:
    URL: "https://www.youtube.com/watch?v=BcS0YiSkV7M"
    Text: "Talk Link"
showToc: False 
disableAnchoredHeadings: false

---


The Tea Time Talks are a series of talks primarily given by the students and faculty studying Artificial Intelligence at the University of Alberta, and provide a comfortable, informal space in which to listen and learn about topics pertaining to machine intelligence and machine learning.

This 2021 Tea Time Talk features Michael Przystupa presenting "Analyzing Neural Jacobian Methods in Applications of Visual Servoing and Kinematic Control".

Abstract: Designing adaptable control laws that can transfer between different robots is a challenge because of kinematic and dynamic differences, as well as in scenarios where external sensors are used. In this work, we empirically investigate a neural networks ability to approximate the Jacobian matrix for an application in Cartesian control schemes. Specifically, we are interested in approximating the kinematic Jacobian, which arises from kinematic equations mapping a manipulator’s joint angles to the end-effector’s location. We propose two different approaches to learn the kinematic Jacobian. The first method arises from visual servoing where we learn the kinematic Jacobian as an approximate linear system of equations from the k-nearest neighbors for a desired joint configuration. The second, motivated by forward models in machine learning, learns the kinematic behavior directly and calculates the Jacobian by differentiating the learned neural kinematics model. Simulation experimental results show that both methods achieve better performance than alternative data-driven methods for control, provide closer approximations to the proper kinematics Jacobian matrix, and on average produce better-conditioned Jacobian matrices. Real-world experiments were conducted on a Kinova Gen-3 lightweight robotic manipulator, which includes an uncalibrated visual servoing experiment, a practical application of our methods, as well as a 7-DOF point-to-point task highlighting that our methods are applicable on real robotic manipulators.


