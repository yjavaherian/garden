---
title: A Tutorial On Multi-View Information Bottleneck
date: 2024-01-30
draft: false
---
In this tutorial we are going to learn about information bottleneck (IB) and the setting in which it is defined. then I will introduce to you a new setting "Multi-View Unsupervised Learning" and we are going to learn how IB can be extended to this setting. this tutorial is based on [Learning Robust Representations via Multi-View Information Bottleneck](https://arxiv.org/abs/2002.07017).
## Information Bottleneck
Consider the standard supervised learning setting; we have an input RV $X$ and a target RV $Y$. we are given a dataset of samples from their joint distribution and are tasked to find a model for predicting $Y$ from $X$. What properties of $X$ and $Y$ affect the performance of our model?
Answer: $I(X;Y)$ is a critical value in this setting. the higher it is the more we are expected to perform and in the limit of $H(Y|X)=0$ there exists a function from $X$ to $Y$ that can predict $Y$ with no error.
but should $I(X;Y)$ be the only thing to measure the difficulty of task with? Well No. Consider two supervised tasks with identical $I(X;Y)$; in practice we are likely to perform better on the task where the relation between $X$ and $Y$ is simpler. Why? because we can look at the problem as finding a point (true conditional distribution $p_d(y|x)$) in a space of probabilities (probability simplex) and simpler relations require smaller search space and thus we need less data to find these points and we are more confident in our finding (bias variance trade off).
Okay having established that, How can we measure the simplicity of the relation between $X$ and $Y$?
Answer: 
One proxy for this is $H(X)$. because (assuming $Y$ is binary) the effective total number of possible relations between $X$ and $Y$ is $2^{H(X)}$. Now out of this insight comes the following:
If we can turn the problem of predicting $Y$ from $X$ into the problem of predicting $Y$ from $Z$ such that $H(Z) < H(X)$ while still $I(Y;Z) = I(Y;X)$ we have turned the original problem into a simpler one and thus we are more likely to achieve a better performance. This is "Information Bottleneck" in a nutshell. Now we reformulate this:

> [!Note] Information Bottleneck Setting
> given two RVs $X$ and $Y$, find a representation for $X$ (called $Z$) such that $Z$ is minimally sufficient for $Y$.

In our formulation we paraphrased the two conditions we enumerated into "minimally sufficient". we now elaborate on this:

> [!Note] Sufficiency
> consider the following Markov chain $Z \leftarrow X \rightarrow Y$; $Z$ is sufficient for $Y$ if $X \rightarrow Z \rightarrow Y$.

by applying Data Processing Inequality (DPI) on both Markov chains we get:
$$
\begin{align*}
I(X;Y) \geq I(Z;Y)\\
I(Z;Y) \geq I(X;Y)
\end{align*}
$$
Therefore sufficiency leads to $I(X;Y) = I(Z;Y)$ and thus sufficiency is what we meant by saying a good representation should keep $I(Z;Y) = I(X;Y)$.

#### More on Sufficiency
Actually $I(Z;Y) = I(X;Y)$ is equivalent to sufficiency; to see this note that $X \rightarrow Z \rightarrow Y$ means $X \bot Y|Z$ and since $Z$ is a representation of $X$ we assume $Z$ is independent of any other random variable conditioned on observing $X$, then we have:
$$
\begin{align*}
I(Y;Z|X) &= 0 \\
H(Y|X) - H(Y|Z,X) &= 0 \\
H(Y|X) - H(Y) - H(Y|Z) + H(Y) + H(Y|Z) - H(Y|Z,X) &= 0 \\
-I(Y;X) + I(Y;Z) + I(Y;X|Z) &=0\\
I(Y;X|Z) &= I(Y;X) - I(Y;Z) 
\end{align*}
$$
Thus  $X \bot Y|Z \Leftrightarrow  I(Y;X|Z) = 0 \Leftrightarrow I(Y;X) - I(Y;Z)$.


> [!Note] Minimal Sufficiency
> A sufficient statistic ($Z$) is a minimal sufﬁcient statistic, if it is a function of every other sufﬁcient statistic $Z'$ . or equivalently $X \rightarrow Z' \rightarrow Z \rightarrow Y$ forms a Markov chain.

Again by applying DPI we get:
$$

I(X;Z) \leq I(X;Z') \quad \forall Z' \text{ being a sufficient statistic}
$$
so minimal sufficient statistic is the one having the lowest $I(X;Z)$ among them which is upper bounded by $H(Z)$. and so this is what we actually meant by trying to get $Z$ such that $H(Z) \leq H(X)$.

### A Note on Minimality
in our discussion so far we have explained the pursuit of minimality based on model complexity and data efficiency. we now give another reasoning based on out of distribution generalization and robustness followed by an example:

the more a representation is independent of $X$ (lower $I(Z;X)$) the more it is robust to changes in $X$ and the more it is likely to generalize beyond train distribution. we provide an intuition for this with an example:
#### Cats and Dogs 😺🐶
consider a dog vs cat classification task where we are given an input image $X$ and tasked to predict the label $Y$ which corresponds to whether the input is an image of a dog or a cat. additionally assume that the training data has the following property:

> cats are more likely to be photographed in indoor environments while dogs are more likely to be photographed in outdoor environments.

but this property is **reversed** **in test distribution**. Now consider two models:
- $M_1$ uses both information of background and object shape to determine the label.
- $M_2$ only relies in object shape.
both these models are going to perform well in training data but it is obvious that $M_2$ is going to perform better in testing data. what separates $M_2$ from $M_1$?  $M_1$ dependence on $X$ is less than  $M_2$.

### Optimization
Knowing more about minimal sufficient statistics we can write the IB objective as:
$$
\text{min } I(X;Z) \text{ subject to } I(Y;Z) = I(Y;X). 
$$
assuming a parametric family of conditional distributions $P_{\theta}(Z|X)$ the Lagrangian of the above is:
$$
\mathcal{L}(\theta,\lambda) = I_{\theta}(Z;X) - \lambda I_{\theta}(Z;Y)
$$
where $\lambda$ controls the trade off between sufficiency and minimality.
There are other ways which we can reformulate the above objective and depending on the scenario, thinking about IB as them can be more useful:
1. Let's write $I(X,Y;Z)$ as following:
	$$
		I(X,Y;Z) = I(X;Z) + I(Y;Z|X) = I(Y;Z) + I(X;Z|Y)
	$$
	Thus $I(X;Z) = I(Y;Z) + I(X;Z|Y)$ and putting this in IB Lagrangian objective we get:
	$$
		\mathcal{L}(\theta,\lambda) = I_{\theta}(Z;X|Y) + (1 -\lambda) I_{\theta}(Z;Y)
	$$
2. Assume $Z$ and $X_{c}$ are both MSS. Now based on the definition of MSS $Z$ must be a function of $X_c$ and vise versa. Hence:
		$$
			H(Z|X_{c})= H(X_{c}|Z) = 0 	
		 $$
	fixing $X_{c}$ , minimizing $H(X_{c}|Z)$ is equivalent to maximizing $I(Z;X_{c})$.
	Additionally following proposition 2.1 of [Achille17](https://arxiv.org/abs/1706.01350) $X$ can be written as $f(X_{c},X_{s})$ where $X_{s}\ \bot\ X_{c}$. so for $Z$ to be a function of $X_c$ means that $Z$ cannot depend on $X_s$ and so $I(Z;X_{s}) = 0$. Now we can write another objective for optimizing a parametric representation based on IB:
	$$
		\text{max } I(X_c;Z) \text{ subject to } I(X_s;Z) = 0. 
	$$
	
	and it's Lagrangian is:
	$$
	\mathcal{L}(\theta,\lambda) = I_{\theta}(Z;X_{c}) - \lambda I_{\theta}(Z;X_{s})
	$$
	we can view $X_{c}$ is the "informative" part of $X$ and $X_s$ as "superﬂuous" part and thus IB tries to increase dependence of $Z$ on $X_c$ and decrease it's dependence on $X_s$.

## Multi View Setting
In IB a principal assumption is that we have a dataset of labeled inputs; however labelling large amounts of data is costly and time-consuming. So in practice we cannot have large labeled datasets. Instead what we have are large unlabeled datasets. There are two scenarios that are of interest to us in such settings:
1. multi-view: our observations are compromised of multiple views. for example we might be interested in classifying a 3D object based on a 2D image of it and we are provided with multiple images of each object taken from different angles. or we want to tag a social media post and we are provided with it's image and author's short description. The underlying assumption in this scenario is that down-stream tasks we are interested in are invariant to the views. to make this more formal we define $X$ to be the original object of interest, $V_i$ to be a view of $X$ and $Y$ a property of interest. then:
	$$
		I(Y,V_1) = I(Y,V_2) = \dots = I(Y,X)
	$$

2. self-supervised: we do not have access to multiple views of $X$ but we are aware of some invariances of label $Y$ w.r.t input in the form of a set of transformations $\mathbb{T}$. meaning that we can create our views by applying transformations present in $\mathbb{T}$ to our observations.

> [!Question] Multi-View IB
> Now a natural question is how one can extend the IB framework into Multi-view setting?

## Method
#### Building Intuition
IB tries to find a representation that while is faithful (sufficient) tries to maximally discard information present in $X$ not helpful for predicting $Y$ (superﬂuous). And so it needs to know $Y$ to answer "is this information predictive of $Y$?". But in unsupervised settings we do not have $Y$. instead what we have are some invariance assumptions about $Y$; in essence these invariances indicate that some parts of $X$ can be discarded for prediction of $Y$. what parts? the parts that are variant. In what follows we build upon this intuition.

We define redundancy to formalize the assumption that all views have the same predictive information regarding $Y$:

> [!Note] Redundancy
>  We say $V_1$ is redundant with respect to $V_2$ for $Y$ if $I(Y;V_1|V_2)=0$. Also we say $V_1$ and $V_2$ are mutually redundant if  $I(Y;V_1|V_2)=I(Y;V_2|V_1)=0$.

The following Corollary formalizes our intuition that variant parts of views can be discarded for prediction of $Y$:

> [!Note] Corollary
> Let $V_1$ and $V_2$ be two mutually redundant views for a target $Y$ and let $Z_1$ be a representation of $V_1$.  If $Z_1$ is sufficient for $V_2$ then $Z_1$ is as predictive for $Y$ as the joint observation of the two views.

based on this Corollary we can now extend IB to the multi-view setting. i.e. a good representation in this setting is the representation found by IB in the supervised task of predicting $V_2$ from $V_1$.

> [!Info]  How should views be? 
>The more two views are different the more they are informative of what information can be discarded and hence the more we lower $I(Z_1;V_1)$. at the extreme the only shared information between views would be the label information and we show that our method reduces to traditional IB.

Now we are in a place to define the optimization objective of Multi-View IB; based on the above ideas we need to focus on $Z_1$ and $Z_2$ that are MSS for predicting $V_2$ and $V_1$ respectively. for $Z_1$ we have:
$$
\mathcal{L_1}(\theta_1,\lambda) = I(Z_1;V_1|V_2) - \lambda I(Z_1;V_2)
$$
where $Z_1 \sim p_{\theta_1}(.|V_1)$. Symmetrically we have:
$$
\mathcal{L_2}(\theta_2,\lambda) = I(Z_2;V_2|V_1) - \lambda I(Z_2;V_1)
$$
averaging the two losses:
$$
    \mathcal{L}(\theta_1,\theta_2,\lambda) = \frac{I(Z_1;V_1|V_2) + I(Z_2;V_2|V_1)- \lambda \biggl( I(Z_1;V_2) + I(Z_2;V_1) \biggl) }{2}
$$
this can be upper bounded by the following loss:
$$
\mathcal{L}_{MIB}(\theta_1,\theta_2,\beta) = -I(Z_1;Z_2) + \beta\ D_{SKL} (p_{\theta_1}(Z_1|V_1) || p_{\theta_2}(Z_2|V_2)) 
$$
where $D_{SKL}$ is the symmetric KL divergence defined as the average of $D_{KL}(p_{\theta_1}(Z_1|V_1)||p_{\theta_2}(Z_2|V_2))$ and $D_{KL}(p_{\theta_2}(Z_2|V_2)||p_{\theta_1}(Z_1|V_1))$.

Intuitively this loss encourages $Z_1$ and $Z_2$ to have high mutual information and thus retain the redundancy present in $V_1$ and $V_2$ while having identical conditional distributions meaning that predicting which view has generated a given $z$ can be done no better than chance and thus $Z_1$ and $Z_2$ have discarded any view specific information.

The symmetric KL can be computed directly when $p_\theta$ s have known density and $I(Z_1;Z_2)$ can be maximized using any sample based differentiable MI lower bound like Jenson-Shannon lower bound or InfoNCE.

Below we have presented the optimization algorithm corresponding to this objective accompanied by an schematic describing the algorithm visually:

![](mib-algo.png)
![](mib-model.png)

> [!Note]  A Note on Single View Setting:
> what can we do when we do not have access to multiple views?
a simple solution is to come up with a set of transformations on data that we believe down-stream tasks would be invariant to. (e.g. rotation of input image in dog/cat classification example).  then we create the required multi-view dataset of our setup as follows:
>1. sample transformations $t_1$ and $t_2$ randomly and independently from set of transformations $\mathbb{T}$.
>2. set $V_1 = t_1(X)$ and  $V_2 = t_2(X)$.
>if our assumption about these transformations is true:
$$
	I(V_1;Y) = I(V_2;Y) = I(X;Y)
$$
and thus these two views are mutually redundant with respect to $Y$.


## Related Work
we employ the information plane to create a holistic view of this method and its relation to other methods of representation learning in multi-view setting.
information plane is a 2D figure where axis represent $I(X;Z)$ and $I(Y;Z)$.
based on IB a good representation should be on the top left corned of the plane.
we can view the proposed method and previous methods as points and curved in this plane:

![](mib-information-plane.png)

- Supervised IB: given label $Y$ optimizing IB objective leads us to the top left corner of the plane. this representation has best predictive performance and is most generalizable.

- Infomax: based on this method good representations should have maximal mutual information with input (right most corner). $Z=X$  trivially satisfies this objective; to prevent this, different methods employ different architectural constraints on $Z=f_\theta(X)$.

- MV-Infomax: these methods are extensions of the Infomax Idea and they try to maximize $I(Z_1;Z_2)$ which can be proved to be a lower bound on $I(Z_1;V_1)$. by maximizing $I(Z_1;Z_2)$ they maintain the sufficiency property but don't put any restrictions on $I(Z_1;V_1)$ which leads to representations having different values of $I(Z_1;V_1)$ ranging from MIB to Infomax.

- $\beta$-VAE: this method finds a latent representation $Z$ that balances compression and informativeness by a hyperparameter $\beta$. the main disadvantage of this method is that informativeness is measured by $I(X;Z)$ and thus when we increase $\beta$, $Z$ becomes more compressed while trying to maintain $I(X;Z)$ but there is no explicit preference to which information about $X$ should be discarded to accomplish the compression of $Z$ and this preference is implicit in the choice of architecture. this means architectural choices can be made such that compression leads to maintaining sufficiency for $Y$ (higher curves in the figure ) or on the other hand opposite of this; such that compression leads to discarding predominantly information pertinent to prediction of $Y$ (lower curves in the figure).

- MIB: our work has the advantage of discarding information like $\beta$-VAE, but explicitly forces $Z$ to maintain sufficiency like Infomax objectives. this leads to discarding of only irrelevant information for predicting $Y$.


## Experiments (Multi View)

#### Sketch-Based Image Retrieval
Dataset. 
The Sketchy dataset consists of 12,500 images and 75,471 hand-drawn sketches of objects from 125 classes. we also include another 60,502 images from the ImageNet from the same classes, which results in total 73,002 natural object images.

Setup.
The sketch-based image retrieval task is a ranking of natural images according to the query sketch. Retrieval is done for our model by generating representations for the query sketch as well as all natural images, and ranking the image by the Euclidean distance of their representation from the sketch representation. The baselines use various domain specific ranking methodologies.  Model performance is computed based on the class of the ranked pictures corresponding to the query sketch. The training set consists of pairs of image $V_1$ and sketch $V_2$ randomly selected from the same class, to ensure that both views contain the equivalent label information (mutual redundancy).

we use features extracted from images and sketches by a VGG architecture trained for classification on the TU-Berlin dataset. The resulting flattened 4096-dimensional feature vectors are fed to our image and sketch encoders to produce a 64-dimensional representation.

![](mib-table1.png)


#### MIR-Flickr
Dataset.
The MIR-Flickr dataset consists of 1M images annotated with 800K distinct user tags.
Each image is represented by a vector of 3,857 hand-crafted image features ($V_1$) while the 2,000 most frequent tags are used to produce a 2000-dimensional multi-hot encoding ($V_2$) for each picture.

Setup.
we train our model on the unlabeled pairs of images and tags.
Then a multi-label logistic classifier is trained from the representation of 10K labeled train images to the corresponding macro-categories.
The quality of the representation is assessed based on the performance of the trained logistic classifier on the labeled test set. 

![](mib-table2.png)

Results.
Our MIB model is compared with other popular multi-view learning models in the above Figure. Although the tuned MIB performs similarly to Multi-View InfoMax with a large number of labels, it outperforms it when fewer labels are available.  Furthermore, by choosing a larger $\beta$ the accuracy of our model drastically increases in scarce label regimes, while slightly reducing the accuracy when all the labels are observed (see right side of Figure).
This effect is likely due to a violation of the mutual redundancy constraint.

A possible reason for the effectiveness of MIB against some of the other baselines may be its ability to use mutual information estimators that do not require reconstruction.  Both Multi-View VAE (MVAE) and Deep Variational CCA (VCCA) rely on a reconstruction term to capture cross-modal information, which can introduce bias that decreases performance.  

## Experiments (Single View)

In this section, we compare the performance of different unsupervised learning models by measuring their data efficiency and empirically estimating the coordinates of their representation on the Information Plane.

Dataset.
The dataset is generated from MNIST by creating the two views, $V_1$ and $V_2$, via the application of data augmentation consisting of small affine transformations and independent pixel corruption to each image.  These are kept small enough to ensure that label information is not effected.  Each pair of views is generated from the same underlying image, so no label information is used in this process.

Setup.
To evaluate, we train the encoders using the unlabeled multi-view dataset just described, and then fix the representation model.  A logistic regression model is trained using the resulting representations along with a subset of labels for the training set, and we report the accuracy of this model on a disjoint test set as is standard for the unsupervised representation learning literature. We estimate $I(X;Z)$ and $I(Y;Z)$ using mutual information estimation networks trained from scratch on the final representations using batches of joint samples $\{(x^{(i)},y^{(i)},z^{(i)})\}_{i=1}^B \sim p(X,Y)p_\theta(Z|X)$.

![](mib-mnist.png)

Results.
The empirical measurements of mutual information reported on the Information Plane are consistent with the theoretical analysis reported in Related word section; models that retain less information about the data while maintaining the maximal amount of predictive information, result in better classification performance at low-label regimes, confirming the hypothesis that discarding irrelevant information yields robustness and more data-efficient representations.
Notably, the MIB model with $\beta=1$ retains almost exclusively label information, hardly decreasing the classification performance when only one label is used for each data point.

### Future Work
- MIB is only applicable to two-view settings and so a prominent future work direction is to extend this idea to multi-view settings with no limit on number of views.

- an important drawback of Information Bottleneck is that it's invariant to one-to-one transformations and thus does not put any constraints on the distribution of learned representation or whether is should be disentangled or not. while other forms of representation learning like VAEs, Normalizing Flows, etc... explicitly regularize representations to admit to a simple prior distribution. I think a promising area of research is combining IB ideas with VAEs in Multi-view setting.

