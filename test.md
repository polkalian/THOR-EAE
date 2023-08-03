We thank the reviewer for the constructive concerns and address them in detail in the following lines.

Q1. Evaluation metrics and experiment design

The reviewer is concerned about the evaluation of InterNav that “in the interactive navigation scenario, typical path length metrics like SPL are no longer good indicators of the amount of time it would take to complete the task.” and “That said, reporting only these above metrics only portrays the benefits of an interactive navigation strategy, while masking away the disadvantages.” and provides a few potential mitigation strategies of evaluation.

A1. We agree that there lacks a good indicator to measure the amount of time for InterNav and add a new metric STS (Success rate weighted by Time Steps), but still persist in the value of evaluating “path efficiency” (i.e. SPL) following prior work of interactive navigation [1,2]. The detailed information is as follows.

First, in InterNav scenarios, SPL shows the ability of agents to clear the path with object interaction, which is crucial especially in cluttered environments. Fei Xia et al. [2] have introduced SPL to interactive navigation as the measurement of path efficiency that “Path Efficiency: how efficient the path taken by the agent is to achieve its goal. The most efficient path is the shortest path assuming no interactable obstacles are in the way.” Consider the situation in that the agent encounters a considerable number of obstacles and there is no reachable path to the goal. The optimal performance should be the agent going directly towards the goal with effective interactions, which is when SPL=1. For InterNav in complex multi-room environments, we aim at improving agent's interactive ability so it can proactively change the environment for better navigation, which makes us believe evaluating SPL is valuable for our task.

Second, we define a new metric to measure the amount of time to complete the task: $STS=\frac{1}{N}\sum_{n=1}^{N}Suc_{n}\frac{L_{n}/grid}{TS_{n}}$, where $L_{n}$ is the shortest path length, $TS_{n}$ is the timesteps agent takes to complete the task, and $grid=0.25m$ is the unit distance of agent moving forward in one step. Thus $L_{n}/grid$ represents the number of timesteps it takes to navigate to the goal by merely moving forward (without spawned obstacles). Since the agent takes atomic actions in AI2-THOR simulator and each of them shares the same amount of time to execute (a timestep), we measure the amount of time with the number of timesteps. STS is higher when the agent accomplishes the task with less time and the ideal situation is that the goal is directly ahead and there is no need for interaction where STS is 1 (it's the most ideal situation for all navigation scenarios so that it's computable for InterNav). As a matter of fact, we train our model following that idea, since our reward shaping $r=r_{success}+\Delta_{dis}-r_{tp}$ encourages interaction that efficiently reduces the goal distance with fewer timesteps, rather than pursuing a shorter trajectory. Thus, both strategies of efficient bypass and effective interaction are rewarded. We report the performance of several models in the table below, and the result shows that STS is a more stringent measure and is able to reflex the performance difference between models. 

|Methods|STS (all)|STS (N$\geq$4)|
|:----:|:----:|:----:|
|PPO|0.122||
|NIE|0.137||
|HER|0.125||
|PPO+intent|0.143||
|CaMP|**0.151**||

In summary, 

Q2. Eval on data subsets. The reviewer suggests that "I would argue in favor of forming two splits of the dataset, one where no non-interactive agent trajectory exists; and one where an optimal non-interactive agent will need to take a longer path".

A2. We appreciate the suggestion and form a split of dataset where non-interactive trajectories (longer than the shortest path) exist to enrich the evaluation. We first calculate the ratio of that split in the whole dataset: 20.5% (overall), 27.4% (1 $\sim$ 2 rooms), 18.6% (3 $\sim$ 5 rooms), 12.4% (6 $\sim$ 10 rooms). Then we report the performance of models (a non-interactive PPO trained on ProcTHOR is included) on the new split in the table below.

|Methods|SR (%)|SPL|STS|FDT|
|:----:|:----:|:----:|:----:|:----:|
|PPO (non-inter)|||||
|PPO|||||
|NIE|||||
|HER|||||
|PPO+intent|||||
|CaMP|||||

Q3.  Concept clarification.

A3. Thanks for the concern. By "causalities from obstacles" we refer to the causal relationships from the obstacle (O) to other causal factors (i.e. O $\rightarrow$ A, O $\rightarrow$ R in Figure 2). By "confounding bias" we refer to the negative phenomenon of $P(R|do(A))\neq P(R|A)$ caused by the confounder (i.e. obstacles in InterNav). And our method is designed to better learn the causality from action to reward (A $\rightarrow$ R). We will revise the concept clarification of our paper for better understanding.

**References**

[1] Zeng, Kuo-Hao, et al. "Pushing it out of the way: Interactive visual navigation." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.

[2] Xia, Fei, et al. "Interactive gibson benchmark: A benchmark for interactive navigation in cluttered environments." IEEE Robotics and Automation Letters 5.2 (2020): 713-720.


Reviewer 2

We thank the reviewer for the constructive criticism. We address the concerns in detail in the following lines.

Q1. Model design explanation. The reviewer wonders “the connection between confounding bias resulting from unmeasurable obstacles and the counterfactual policy design”, “how the weighted-sum of action logits from the sub-control policies can fully capture the policies' intents and uncover the causality depicted in Fig. 2(a)”, and argues that “a more straightforward approach would be to recursively use the Master Policy to obtain intents and provide feedback to the Master Policy”.

A1. We appreciate the questions and further clarification is detailed as follows.

First, counterfactual decision-making theoretically addresses the problem of learning sub-optimal policy caused by confounding bias from unmeasurable obstacles. Obstacles in InterNav scenario can be viewed as an unobserved confounder (UC) since they influence both the decision-making of actions and the generation of rewards. For instance, the agent may decide to take object interactions when encountering an obstacle (O $\rightarrow$ A). Here, obstacles (O) serve as a mediator from state to action (S $\rightarrow$ A). The causality of A $\rightarrow$ R is confounded by UC, leading to poor value estimation which likely results in sub-optimal policy in RL training. And it can be theoretically proved that a counterfactual policy considering intent obtains more value than a standard policy when there is UC [1]. 

In Section 4, we apply counterfactual policy to a hierarchical decision framework. In addition to addressing UC, learning counterfactual policy also addresses the indirect reward problem of master policy by providing it with information about the low-level decision-making through integrated intent.

Second, we believe the sum of actions from sub-control policies weighted by master policy’s decision can fully represent agent’s hierarchical intent, since it contains agent’s intent on each atomic action and the full distributional information of four policies. Since the primary definition of intent is “action before execution” that $I=i_t=f_i(s_t,o_t)$, we don’t see the rationale of using hidden features from GRU as intent instead of action logits. The relationship between intent and confounding bias is elaborated above.

Third, 
