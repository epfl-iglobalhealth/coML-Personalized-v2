# Semester project on private and personalized ML
semester project felix grimberg - private robust and personalized ML

Read about the aim and objectives of this project in the [report](Report - 200331.pdf).

In this repo, we investigate methods for automatic data selection based on the distance between gradients in SGD-based optimization algorithms.

### Setting:

We consider a network of participants $u_i$, each collecting samples from an underlying distribution $\mathcal{D}_i$.
One participant $u^*$ is called the user. This participant wishes to perform an inference task, such as (regularized) linear or logistic regression, to gain knowledge about the distribution $\mathcal{D}^*$ from which they collect data.
The user $u^*$ further believes that _some, but not all,_ of the other participants $u_i$ collect sufficiently interoperable data from sufficiently similar underlying distributions, that the samples $\mathcal{S}_i$ collected by them could help reduce the _true loss_ of $u^*$'s model on $\mathcal{D}^*$, if the samples $\mathcal{S}_i$ are included in the training of the regression model.
Our task is to define an algorithm which can help the user $u^*$ minimize the expected _true error_ of the fitted regression model on $\mathcal{D}^*$ by selecting a subset of participants whose collected samples are used during training.

As an example, the participants could be individual hospitals dispersed across one or several regions. Doctors could have reasons to expect that the same set of symptoms is more strongly associated with one diagnosis for the patients of one hospital, and with another diagnosis for the patients of a different hospital.
As a layman example, it seems conceivable that diseases caused by poor sanitation are more likely to occur in poor rural regions, while drug abuse is more frequent in more affluent urban settings - while causing similar symptoms. This is not to say that the symptoms are indeed similar.
Knowing this, a medical professional from one hospital might want to train a diagnostic model, recognizing that they would benefit from enhancing the limited number of samples collected at their own hospital with samples collected at a selected subset of other hospitals.

### Why we investigate methods based on the distance between gradients in SGD-based optimization algorithms:

We say that a distribution $\mathcal{D}_i$ is _similar to $\mathcal{D}^*$ with respect to the specific inference task at hand_ (in short: _similar_), if the weights of each distribution's true model for this inference task are similar.

- [ ] We might want to define a metric for that notion of _similarity_ $\uparrow$

Recall that, given an inference task defined by:
- a class of models $\mathbb{M}$ and
- a loss function $\mathcal{L} (y, \hat{y})$,
the true model $\mathcal{M}_{i}^{true}$  minimizes the expected loss $\mathcal{R} \left( \mathcal{D}_i, \mathcal{M} \right)$ over the distribution $\mathcal{D}_i$:

$ \mathcal{R} \left( \mathcal{D}_i, \mathcal{M}_{i}^{true} \right) \triangleq \underset{(\mathbf{x}, y) \in \mathcal{D}_i}{E} \left[ \mathcal{L} \left( y, \mathcal{M}_{i}^{true} \left( \mathbf{x} \right) \right) \right] \leq 
\mathcal{R} \left( \mathcal{D}_i, \mathcal{M}_{i}^{other} \right)
\forall \mathcal{M}_{i}^{other} \in \mathbb{M} $

Let $\mathcal{M}^{tentative} \in \mathbb{M}$ be a tentative model which performs very suboptimally on two **similar** distributions $\mathcal{D}_i$ and $\mathcal{D}_j$. In particular let:

$ \textbf{min} \left( \mathcal{R} \left( \mathcal{D}_i, \mathcal{M}^{tentative} \right),  \mathcal{R} \left( \mathcal{D}_j, \mathcal{M}^{tentative} \right) \right) \gg \textbf{max} \left( \mathcal{R} \left( \mathcal{D}_i, \mathcal{M}^{true}_j \right),  \mathcal{R} \left( \mathcal{D}_j, \mathcal{M}^{true}_i \right) \right)$

Further, let $\mathbf{g}_i$ and $\mathbf{g}^*$ be the gradients of the loss function $\mathcal{L} \left( y_i, \mathcal{M}^{tentative} \left( \mathbf{x}_i \right) \right)$, respectively $\mathcal{L} \left( y^*, \mathcal{M}^{tentative} \left( \mathbf{x}^* \right) \right)$, for a sample $(y_i, \mathbf{x}_i \in \mathcal{D}_i)$, respectively $(y^*, \mathbf{x}^* \in \mathcal{D}^*)$, with respect to the model weights.
Then it follows that, in expectation over all possible samples from each distribution, $\mathbf{g}_i$ and $\mathbf{g}^*$ will be similar as well.

- [ ] We might want to prove that $\uparrow$, for which we would also need to be more rigorous about what we mean by the two gradients being similar.
- [ ] We haven't shown that the gradients of two _more similar_ distributions will be more similar on average than the gradients of two _less similar_ distributions.
- [ ] Here we assume that $\mathcal{M}^{tentative}$ performs very suboptimally on both distributions. What happens as $\mathcal{M}^{tentative}$ approaches, let's say, the _true model of the joint distribution_? Clearly, the gradients will begin to diverge. Can we formalize this in some way? In fact, I believe that the true model of the joint distribution is such that the sum $\mathbf{g}_i + \mathbf{g}^*$ adds up to $\mathbf{0}$ in expectation.
- [ ] Related to the previous point: Maybe the aggregated similarity estimator (cf. below) should experience some form of decay, if we expect the (expected) gradients to diverge as the model gets better? But then, if we do enough SGD steps, all similarities would end up decaying at some point and we would eventually end up with the local model again. So where do we find the compromise here?
- [ ] Is there a canonical symbol to denote the _true error_ of a model? That's the correct name for what I'm defining here, right?
- [ ] I'm almost certain that _true model_ is not the correct term for the model that minimizes the true error over all models in a class of models. What's the correct term?
- [ ] We might want to distinguish between the norm and the angle of the gradients. Though a gradient that points very far or not-far-at-all in the right direction still is not useful and certainly does not hint at _similar underlying distributions._

It is therefore our intention to invent an estimator for the (so far vaguely defined) similarity of two distributions based on the distance between gradients in SGD-based optimization algorithms involving samples taken from these two distributions.
It is clear that this estimator must be aggregated over a number of SGD or minibatch steps due to the random nature of the SGD / minibatch gradient.

### Methods:

This whole thing about similar distributions leading to similar gradients holds _in expectation_, but the individual gradients can randomly vary a lot if the minibatch size is small.
It is thus clear that we must come up with a similarity estimator that's aggregated over time.

The following could be a rough sketch of the algorithm:
1. Perform a certain number of steps with all participants until we've aggregated a stable enough similarity estimator.
2. Select a subset of the available data sets, potentially weighing the contribution of each data set according to the estimated similarity of its underlying distribution.
3. Train the model on the selected data sets until some stopping criterion (convergence, number of communication rounds, ...) is reached. Step 2 could potentially be repeated at regular intervals throughout step 3.

There is a balance to be struck between the number of communication rounds needed to perform steps 1 & 3 satisfactorily, and the requirement of _communication efficiency_.

I don't know whether weight quantization and model sparsification are viable strategies to achieve communication efficiency in the training of linear models.
