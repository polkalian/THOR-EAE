**Reviewer 1**

We thank the reviewer for the constructive concerns and address them in detail in the following lines.

Q1. Evaluation metrics and experiment design

The reviewer is concerned about the evaluation of InterNav that “in the interactive navigation scenario, typical path length metrics like SPL are no longer good indicators of the amount of time it would take to complete the task.” and “That said, reporting only these above metrics only portrays the benefits of an interactive navigation strategy, while masking away the disadvantages.” and provides a few potential mitigation strategies of evaluation.

A1. We agree that “time of completing the task” is also an important evaluation indicator for InterNav. In our scenario, by calculating the standard time cost of each action (including movement and manipulation), a time cost metric can also be evaluated in our experiments, which can be regarded as a time-measurement variant of the SPL metric. Also, considering most previous InterNav works [1,2] report the SPL, in order to compare with those works directly, we’ll report both time and path length metrics in the following response and final version.

First, we evaluate an additional metric STS (short for Success rate weighted by Time Steps) to measure the time cost of task completion: $STS=\frac{1}{N}\sum_{n=1}^{N}Suc_{n}\frac{L_{n}/grid}{TS_{n}}$, where $L_{n}$ is the shortest path length, $TS_{n}$ is the timesteps agent takes to complete the task, and $grid=0.25m$ is the unit distance of agent moving forward in one step. Thus $L_{n}/grid$ represents the number of timesteps it takes to navigate to the goal by merely moving forward (without spawned obstacles). Since the agent takes atomic actions in AI2-THOR simulator and each of them shares the same amount of time to execute (a timestep), we measure the time cost with the number of timesteps. STS is higher when the agent accomplishes the task with less time and the ideal situation is that the goal is directly ahead and there is no need for interaction where STS is 1 (it's the most ideal situation for all navigation scenarios so that it's computable for InterNav). As a matter of fact, we train our model following that idea, since our reward shaping $r=r_{success}+\Delta_{dis}-r_{tp}$ encourages interaction that efficiently reduces the goal distance with fewer timesteps, rather than pursuing a shorter trajectory. Thus, both strategies of efficient bypass and effective interaction are rewarded. We report the performance of several models in the table below, and the result shows that STS is a more stringent measure and is able to reflex the performance difference between models. 

|Methods|STS (all)|STS (N$\geq$4)|
|:----:|:----:|:----:|
|PPO|0.122|0.054|
|NIE|0.137|0.088|
|HER|0.125|0.051|
|PPO+intent|0.143|0.086|
|CaMP|**0.151**|**0.092**|

Second, in InterNav scenarios, SPL is an indicator of agents' ability of object interaction, which is crucial especially in cluttered environments. Fei Xia et al. [2] have introduced SPL to interactive navigation as the measurement of path efficiency that “Path Efficiency: how efficient the path taken by the agent is to achieve its goal. The most efficient path is the shortest path assuming no interactable obstacles are in the way.” Although the most efficient path may not correspond to the least time cost, it indicates the most effective interaction. For InterNav in complex multi-room environments, we aim at improving agent's interactive ability so it can proactively change the environment for better navigation, rather than limiting its strategic choices to its capacity.

Q2. Eval on data subsets. The reviewer suggests that "I would argue in favor of forming two splits of the dataset, one where no non-interactive agent trajectory exists; and one where an optimal non-interactive agent will need to take a longer path".

A2. We appreciate the suggestion and form a split of dataset where non-interactive trajectories (longer than the shortest path) exist to enrich the evaluation. We first calculate the ratio of that split in the whole dataset: 20.5% (overall), 27.4% (1 $\sim$ 2 rooms), 18.6% (3 $\sim$ 5 rooms), 12.4% (6 $\sim$ 10 rooms). Then we report the performance of models (a non-interactive PPO trained on ProcTHOR is included) on the **new split** in the table below. It's interesting to find that PPO without interaction achieves better STS (0.201) compared with PPO (0.181), although it obtains lower SR on the non-interactive set (47.4%) and the whole set (21.5%). It indicates that the strategy of object interaction may cost unnecessary time in uncrowded environments and the agent needs to balance the efficiency and efficacy during the task.

|Methods|SR (%)|SPL|STS|FDT|
|:----:|:----:|:----:|:----:|:----:|
|PPO (non-inter)|47.4|0.309|0.201|4.62|
|PPO|51.5|0.306|0.181|3.44|
|NIE|58.8|0.345|0.188|3.01|
|HER|51.9|0.316|0.176|3.40|
|PPO+intent|70.4|0.390|0.222|2.38|
|CaMP|**72.3**|**0.407**|**0.236**|**2.05**|

Q3.  Concept clarification.

A3. Thanks for the concern. By "causalities from obstacles" we refer to the causal relationships from the obstacle (O) to other causal factors (i.e. O $\rightarrow$ A, O $\rightarrow$ R in Figure 2). By "confounding bias" we refer to the negative phenomenon of $P(R|do(A))\neq P(R|A)$ caused by the confounder (i.e. obstacles in InterNav). And our method is designed to better learn the causality from action to reward (A $\rightarrow$ R). We will revise the concept clarification of our paper for better understanding.

**References**

[1] Zeng, Kuo-Hao, et al. "Pushing it out of the way: Interactive visual navigation." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.

[2] Xia, Fei, et al. "Interactive gibson benchmark: A benchmark for interactive navigation in cluttered environments." IEEE Robotics and Automation Letters 5.2 (2020): 713-720.


**Reviewer 2**

We thank the reviewer for the constructive criticism. We address the concerns in detail in the following lines.

Q1. Model design explanation. The reviewer wonders “the connection between confounding bias resulting from unmeasurable obstacles and the counterfactual policy design”, “how the weighted-sum of action logits from the sub-control policies can fully capture the policies' intents and uncover the causality depicted in Fig. 2(a)”, and argues that “a more straightforward approach would be to recursively use the Master Policy to obtain intents and provide feedback to the Master Policy”.

A1. We appreciate the questions and further clarification is detailed as follows.

First, counterfactual decision-making theoretically addresses the problem of learning sub-optimal policy caused by confounding bias from unmeasurable obstacles. Obstacles in InterNav scenario can be regarded as an unobserved confounder (UC) since they influence both the decision-making of actions and the generation of rewards. For instance, the agent may decide to take object interactions when encountering an obstacle (O $\rightarrow$ A). Here, obstacles (O) serve as the mediator from state to action (S $\rightarrow$ A). The causality of A $\rightarrow$ R is confounded by UC, leading to poor value estimation which likely results in sub-optimal policy in RL training. And it can be theoretically proved that a counterfactual policy considering intent obtains more value than a standard policy when there is UC [1]. 

In Section 4, we apply counterfactual policy to a hierarchical decision framework. In addition to addressing UC, learning counterfactual policy also addresses the indirect reward problem of master policy by providing it with information about the low-level decision-making through integrated intent.

Second, we believe the sum of actions from sub-control policies weighted by master policy’s decision can fully represent agent’s hierarchical intent, since it contains agent’s intent on each atomic action and the full distributional information of four policies. Since the primary definition of intent is “action before execution” that $I=i_t=f_i(s_t,o_t)$, we don’t see the rationale of using hidden features from GRU as intent instead of action logits. The relationship between intent and confounding bias is elaborated above.

Third, we find the idea of extending the recursive feedback to multiple levels interesting since it may explore the effect of recursive intent, namely the intent generated based on a priori intent. And we are training new models based on the baseline of PPO to study how a recursive intent may help the policy learning. Nonetheless, we believe implementing agent’s intent with an intent policy is reasonable and it’s flexible for us to utilize old intent from iterations behind to balance the policy exploration.

Q2. Missing details and confusions.

A2. Thanks for pointing them out and we will reply to them item by item as follows.

o) When taking Push/Pick actions, the object to interact with would be the closest pushable/pickable (predefined according to category) and observable (within 1.25m) object.

o) The amount of force applied on the object during the Push action is 100 Newton.

o) The dimension of intent embedding is 12, equivalent to the size of our action space.

o) The CNN is implemented as a simple CNN and the number of GRU layers is 1, which is in line with prior work [2] and the default setting of AllenAct.

o) “Epochs with a rollout of data” refers to the number of update iterations using a rollout of data in PPO.

o) In the HRL baseline, the master policy calls one of the sub-policies (with the same splits of action space of our model) at each step and the action is output by the sub-policy.

o) It’s a typo and both should be step penalty $r_{sp}=0.01$.

o) The reward for interactive auxiliary tasks should be $r_{inter}=r+r_{as}-r{af}$, where $r=r_{success}+\Delta_{dis}-r_{sp}$ and $r_{success}$ is obtained when the goal of interactive task is achieved (taking Done when the obstacle is cleared).

**References**

[1] Zhang, Junzhe, and Elias Bareinboim. Markov decision processes with unobserved confounders: A causal approach. Technical report, Technical Report R-23, Purdue AI Lab, 2016.

[2] Zeng, Kuo-Hao, et al. "Pushing it out of the way: Interactive visual navigation." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2021.


**Reviewer 3**

We appreciate the detailed questions and reply to them in the following lines.

Q1. Action space and positioning of objects.

A1. Our CaMP agent is trained using exactly the same action space as other baselines. In particular, the agent makes only 90-degree turns when taking *RotateRight* and *RotateLeft*, and rotates the camera elevation angle by ±30 degrees with *LookUp* and *LookDown*. The turning angles shown in Figure 5 are just for visualization by combining the trajectories of several adjacent steps (e.g. *MoveAhead*+*RotateLeft*+*MoveAhead*="move diagonally"). In section 5.1, we mention "adjacent nodes" referring to different doors on the path of cross-room navigation. As for the position of objects, all obstacles are spawned on the reachable points of agents, which are distributed on a grid network with a grid size of 0.25m.

Q2. Object dynamics in ProcTHOR.

A2. The interactive attributes of objects are decided according to their object categories in the ProcTHOR simulator. For example, different assets of *Table* are all moveable and not pickable, regardless of their mass and size. We speculate that CaMP policy learns about those attributes by generalizing based on both visual appearance from image and depth sensors and prior experience of interaction. In particular, assume the representation of obstacles can be implicitly formulated as $H_{O}=F_{S \rightarrow O}(S, \theta)$, which is produced with function $F_{S \rightarrow O}$ (parameterized by $\theta$) given the observation $S$. We believe that with the help of counterfactual exploration, the agent can accumulate more useful experience for the optimization of $\theta$.

Q3. Details of invoking and terminating interaction subtasks.

A3. The master policy consists of a GRU and a MLP, and keeps receiving observations when an interaction sub-task is running, yet the output of the master policy will not influence the behavior of the agent. In Figure 3 we use green arrows to denote the call and control procedure of multi-policy collaboration and we will modify it to further clarify the transition of control.

Q4. Learning for the intent policy parameters.

A4. We keep the parameters of the intent policy frozen in between syncs since the intent produced by the intent network is just for replicating the decision of the master policy.

Q5. Sim-to-real transfer and application in a real-world setting. The reviewer sets a few assumptions including the hierarchical architecture, practical low-level policies on robots, and simulated training of the master policy. Then the reviewer is concerned about "chaining real-world nav and push subtasks" with the master policy.

A5. We appreciate the decent concern and agree with the assumptions. Overall, we believe it's practical to transfer our method first to comprehensive, manipulation-included,  simulated tasks and then to real-world settings to overcome the sim-to-real challenge. It's true that the sim-to-real gap of object interaction supported by ProcTHOR is quite big compared with that of navigation. Thus we speculate the two main challenges of applying our method: First, the increasing amount of time the agent takes to accomplish interactive sub-tasks, leading to sparse feedback for decision-making of the master policy. Second, object manipulation and interaction dynamics are more complex to learn with limited training data. For the former, we plan to develop two pipelines of training (i.e. high-level and low-level) as future work, where two sets of MDP, as well as PPO, are operated in separate time abstraction, so that the returns of the whole interactive sub-task are compressed to that of one high-level timestep for the master policy. For the latter, recently large-scale training for object manipulation with robotic arms becomes possible due to the development of simulators [1,2]. Thus we believe more comprehensive tasks with general purposes (e.g. rearranging a cluttered room [3]) and complex interactions (including object manipulation) should be studied before applying algorithms to real-world interactive tasks.

**Q6. Intuition and conceptual value of the intent component**. The reviewer questions about "How does the causal policy architecture "boost" this exploration", "what context the intent is providing about obstacles", and "What is the essential difference with your causal policy" compared with a stochastic policy.

A6. We reply to the questions as follows.

First, training based on intent boosts the exploration of new strategies by accumulating new experiences counterfactual to that from the original policy. Representing an experience with a tuple $<s_t,a_t,r_t>$, the experiences collected with a standard policy follow a certain distribution where 

**References**

[1] Ehsani, Kiana, et al. "Manipulathor: A framework for visual object manipulation." Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. 2021.

[2] Xiang, Fanbo, et al. "Sapien: A simulated part-based interactive environment." Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition. 2020.

[3] Weihs, Luca, et al. "Visual room rearrangement." Proceedings of the IEEE/CVF conference on computer vision and pattern recognition. 2021.
