# Discrete Automation PLC Programming of a Production Line using standard methodologies

## Description of the Task

The system shown in the figure below is used to deliver the pallets to the workingstations (WS1..3).

<a href="https://drive.google.com/uc?export=view&id=1qaBC0UiOrp2oQ_UWMFE3qBsGo7X-_MGB"><img src="https://drive.google.com/uc?export=view&id=1qaBC0UiOrp2oQ_UWMFE3qBsGo7X-_MGB" style="width: 500px; max-width: 100%; height: auto" title="Click for the larger version." /></a>

The system is composed of a conveyor loop and three workstations joined with it. The conveyor is used to deliver pallets to the workstations. The pallet is introduced to the system at WS1, then each pallet must visit WS2 and WS3. The pallet that has visited all the workstations is removed from the system at the WS1. Each workstation can process 1 pallet at the time. It takes 5 seconds for a workstation to process the pallet. If the workstation is busy processing the pallet and a new pallet comes to its location needing also the service of that particular workstation, then that new pallet continue on the conveyor loop to come to the workstation on the next cycle - as it circulates the conveyor belt to appear in front of that workstation again. It is does not matter in which order the WS are visited - important is that all of them will be visited. The conveyor can be seen as a dynamic buffer that can have pallets circulating until a pallet has visited all the workstations.

The system should have at least following outputs:
* Motor - the motor to control the conveyor. (BOOL) In general should be always on, as the pallets are stopped by the stoppers at the WS.
* Stoppers S1..3 - to stop a pallet at the corresponding workstation. (BOOL)
* Board available BA1..3 - to signal to the corresponding workstation that the pallet has arrived. (BOOL)


The system should have at least the following inputs:
* Locked by WS L1..L3 - WS signals that it already has the pallet for the processing. (BOOL)
* Presence sensors P1..3 - sensors telling that there is a pallet in the corresponding location in front of a WS. (BOOL)
* Pallet_in - A button to introduce a pallet to the system at WS1 position.

With respect to BAx and Lx signals the logic can look as follows: A pallet arrives at WSx, the program checks Lx. a) If Lx is true the pallet continues by the conveyor to the next WS. b) If Lx is false and the pallet needs the service of that WS, then BAx signal goes on. The BAx signal should go off once Lx goes on, which means that the WSx has taken the pallet. The pallet is then in the WSx (for 5 seconds). Other pallets can bypass the WSx holding a pallet. Then, after the processing, the pallet should be returned to the conveyor belt and Lx goes off. (The pallet can be returned from the WS to the conveyor belt, if and as there is no any other pallet - Px and BAx inputs are off.) 

A memory structure that monitors if the pallet was tended at all the workstations would also have to be implemented.

----------------------------------------------------------------------------------------------------------------------------------------


# Solution

This problem is solved using two methodologies namely using a State Chart Methodology and Flow Chart Methodology. Below is the final GUI of the Solution

<a href="https://drive.google.com/uc?export=view&id=1XDvJxu2HliAcw9uHXVgDzL0TZBolxFH4"><img src="https://drive.google.com/uc?export=view&id=1XDvJxu2HliAcw9uHXVgDzL0TZBolxFH4" style="width: 500px; max-width: 100%; height: auto" title="Click for the larger version." /></a>

## State Chart Methodology

In this methodology, we have to create and define states where the system can be. From the definition of the problem, 4 different external states was chalked out, as we can see in the state diagram below.

#### External States:

* IDLE: state where the system does nothing until Start button is pressed;
* START: state where the system waits for Pallet_in to be pressed;
* WS1: state where workstation 1 is processing the pallet;
* Conveyor:  this state means that there are pallets inside the system, in some position. In this state, we will check the location of each pallet and, depending on the configurations of the inputs, we will carry out some actions: change states of the pallets, change values of the outputs…etc.



<a href="https://drive.google.com/uc?export=view&id=1MbztRWO5wwtd0o0ZF8CN_EJ3ffGsQxcn"><img src="https://drive.google.com/uc?export=view&id=1MbztRWO5wwtd0o0ZF8CN_EJ3ffGsQxcn" style="width: 500px; max-width: 100%; height: auto" title="Click for the larger version." /></a>

In the Conveyor state, we have a lot of other states where the system can jump, depending on the inputs. Those states will be called Internal States. We call them that because each pallet will have its own Internal States. Each pallet will have the information about where it is and what is happening to it. For example, pallet 1 can be in one position in the conveyor and pallet 2 can be inside of a
workstation being processed. This kind of information will be saved inside the pallet structure.

#### Internal States:

* CHECK2: When the pallet is introduced, it is processed by workstation and then it goes to position 0 (position 1 in the visualization. This difference is because the first element of an array has index 1). In this position the pallet has to wait until P2 is pressed. When the presence sensor 2 in ON, that means that the pallet is now in front of workstation 2. Now the position of the pallet is position 1 (position 2 in the visualization). If the workstation 2 is working or this pallet already has the service from it, we jump to state PALLET PASS2;

* PALLET PASS2: this state exists to represent the pallet passing through the stopper 2. In other words, if the pallet reaches this state, it means that that pallet has to pass through the stopper and because of that, the stopper 2 goes down and after an amount of time, goes up again to stop the next pallet. We define 2 seconds for this to happen. Summing, in this state, the stopper 2 goes down during 2 seconds and then he goes up again. The pallet is now in the position 2 (position 3 in the visualization).

* PICK-UP2: this state represents the workstation picking up the pallet to be processed inside of it. We define the picking up process to take also 2 seconds. During this 2 seconds, BA1 is activated meaning that there is a pallet to be picked up. After 2 seconds, the pallet is position 7 (position 8 in the visualization).

* WS2: in this state, the pallet is being processed. This takes 5 seconds. After this time, the pallet goes to position 2 (position 3 in the visualization), and the workstation is now free for a new pallet.


The states CHECK3, PALLET PASS3, PICK-UP3, WS3, CHECK1 and PICK-UP1 are very similar to CHECK2, PALLET PASS2, PICK-UP2 AND WS2 but for different conditions of jumping, as we can see in the State Diagram.

## Flow Chart Methodology

For this methodology, instead of state we have functions and transitions. As we did for the state chart methodology, in terms of functions, there are 6 main functions, as we can see in the flowchart below

<a href="https://drive.google.com/uc?export=view&id=1QGYeYNDuRSa5ppBVVFJOWuJDqGfiqPLg"><img src="https://drive.google.com/uc?export=view&id=1QGYeYNDuRSa5ppBVVFJOWuJDqGfiqPLg" style="width: 500px; max-width: 100%; height: auto" title="Click for the larger version." /></a>
1QGYeYNDuRSa5ppBVVFJOWuJDqGfiqPLg

#### External Functions:

* Start: functions that is activated when Start button is pressed;
* Pallet_in: function that is activated when Pallet_in button is pressed
* L1: function that is activated when the workstation 1 is free
* Insert: function that is going to insert a pallet in the system 7
* WS1: function that means that the workstation 1 is processing the inserted pallet
* RUN: function similar to the state named Conveyor in the state chart methodology
* Pallet_in 2: function that is true when last internal function is done

 

After RUN function we have a lot of internal functions that are going to be inside of the structure of
each pallet.
 
The memory structure is the same as for the state chart methodology. The difference is that instead
of saving the information about internal states, we save information about internal functions and
transitions. Some of this functions and transitions are made in just one state of the state chart
methodology. That’s why there are more functions than states. Reading the flowchart, it becomes
easy to see what each transition and function does.

### Pallet Tracking

It is imperative to know the location of each of the pallets for the logic to work. Now I will go in detail the logic how the pallet is tracked in this example.

​

When a presence sensor is pressed, in order to know what pallet should go to the position in front of that workstation, first we check what are the pallets that are in the position before that. After this step, we have to know what is the pallet that comes in the first place. This is a difficult thing to know. We choose to save this information inside the structure of each pallet. In other words, we have created 2 more parameters inside the structure of the pallet, that are the same as ‘pointers’. So, each pallet knows what is the pallet that is behind and in front of it. These parameters will change when a pallet is introduced into the systems, when a pallet passes in front of a pallet that is being processed by a workstation and when a pallet is removed from the system. We can see these two parameters in the visualization. There is also 1 more parameter that we have created to know what is the number of the pallet.
When a pallet is introduced to the system, the structure in the array that has the index of number of pallets introduced, is going to be activated. When there is more than 1 pallet in the system, and a pallet (not the last one that was introduced) is removed, there will be a free space in the array. Because we introduced pallet in the number of pallets introduced, we will continue to have a free space in the
array. This is not good because this position can reach the last position of the array and then the program will break. In order to solve this problem, we decided that, when a pallet is removed from the array, the position in the array of the structure of this pallet will be occupied by the last structure that is active in the array. After doing this, we can activate structures in the position number of pallets introduced minus number of pallets removed. This preserves the space in the array. Having this kind of structures and management of space in the array, doing the management logic for pallets tracking is easy. We just have to know where is the structure of each pallet inside the array and print the position of it.
