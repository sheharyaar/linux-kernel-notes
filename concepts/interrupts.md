- [Introduction](#introduction)
- [Types of Interrupts](#types-of-interrupts)
- [Interrupt Controllers](#interrupt-controllers)
- [Interrupt Flow](#interrupt-flow)
- [Top Halves](#top-halves)
- [Bottom Halves](#bottom-halves)

## Introduction

- event that alters the normal flow of a program
- can be generated by hardware devices or by the CPU itself
- physically produced by **electronic signals** generated by hardware and directed to the input pins on an `interrupt controller`.

## Types of Interrupts

- Based on the source of interrupt : 
	- Synchronous → Generated by executing an instruction (Eg : [syscalls](./syscalls/syscalls.md), Divide by Zero)
	- Asynchronous → Based on external event (Eg : Key presses on keyboard)
- Based on ability to temporarily disable :
	- Maskable → can be ignored; it is signaled via **INT** pin
	- Non-Maskable → cannot be ignored ; it is signaled via **NMI** pin


## Interrupt Controllers

> An interrupt controller is a simple chip that multiplexes multiple interrupt lines into a single interrupt line on the processor.

- when an interrupt occurs, the current flow of execution is stopped and the **interrupt handler** runs (unless interrupts are disabled for **critical sections**)
- Each interrupt has a unique value assigned to it so that interrupts from 2 different devices can be differentiated. These values are called `Interrupt Request (IRQ) lines`. 

## Interrupt Flow

TODO: add flow excali svg here

## Top Halves

## Bottom Halves

TODO: add backlinks here

Their are 4 ways to defer work in Bottom Half 
1. softirq 
2. tasklet 
3. [workqueue](../core-apis/core-utilities/workqueue.md) (replacement of task queues) 
4. kernel Timer