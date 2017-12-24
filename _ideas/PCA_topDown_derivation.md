I think there should be slight corrections in experiment.py related to how we shift to mean and project vectors from pixel space to eigenspace.

Corrections:
1. Shift to mean:
'experiment.py' calculates $$\mu$$ (mean) for both training and testing. I think $$\mu$$ should be common between training and testing, as $$\mu$$ is a property of transformation. So it should just be calculated from training set and applied to test set directly. That is,
mu = ps6.get_mean_face(Xtrain)
...
# testing
mu = ps6.get_mean_face(Xtest)
Xtest_proj = np.dot(Xtest - mu, eig_vecs)

2. Normalize projected vector in eigenspace by standard deviations ($$\sqrt{eigenvalue}$$'s) in corresponding dimensions.
This might be tricky to understand, as lectures have also not pointed this out.
To motivate the claim that this should be done, please see my comparison plot.

Orange line is straight as per 'experiment.py' (with $$\mu$$ correction as explained above).
Blue is when I also normalize with $$\sigma$$'s (standard deviations) in corresponding dimensions.
### <Insert picture here>

TL;DR
we were earlier just doing (after taking top 'k' eigenvectors)
$$A^T.(X-\mu)$$ ... i.e. simply projecting $$(X-\mu)$$ onto eigenvectors.
But we should do,
$$\Lambda^{-1/2}.A^T.(X-\mu)$$ ... i.e. projecting $$(X-\mu)$$ onto eigenvectors and normalizing by $$\Lambda^{1/2}$$
where,
A: Matrix of eigenvectors, such that $$A^T.A = I$$
$$\Lambda$$: Diagonal Matrix of eigenvalues.

Proof:
Now why would this make sense? Intuitively, consider 2 probability distributions A and B and lets say both are Gaussians.
Case A:
$$\mu$$=0, $$\sigma$$=10
a point $$P_A$$=20 is 2 $$\sigma$$'s away from the mean of distribution A.

Case B:
$$\mu$$=0, $$\sigma$$=100
a point $$P_B$$=100. is 1 $$\sigma$$ away from the mean of distribution B.

Question is: which point is closer to $$\mu$$'s of respective distributions?
If you just consider Euclidean distance, you will say $$P_A$$. But that would be wrong as we are interested in how far a point is from the distribution. So in that sense, $$P_B$$ is closer to the distribution.

Now, what does it have to do with PCA?
You might have noticed that PCA minimizes the Mean Squared Error (MSE) between original data (image in our case) and reconstructed data. It is well know that MSE minimization (like in linear regression) assumes a Gaussian prior on data.
Now since we are dealing with multiple dimensions, we need to consider multi-variate Gaussian distribution, which has following form.
$$N(X | \mu, \Sigma) = \frac{1}{\sqrt{det(2.\pi.\Sigma)}} * e^{\frac{-(X-\mu)^T.\Sigma^{-1}.(X-\mu)}{2}}$$
Important consideration that comes out from multi-variate Gaussian distribution is Mahalanobis distance =  $$\sqrt{(X-\mu)^T.\Sigma^{-1}.(X-\mu)}$$.
This is a dimensionless quantity and incorporates ($$X-\mu$$) normalization by $$\Sigma^{1/2}$$ as I show below.

$$(X-\mu)^T.\Sigma^{-1}.(X-\mu) = (X-\mu)^T.(\Sigma^{-1/2}).(\Sigma^{-1/2})^T.(X-\mu)$$
(where, $$\Sigma^{-1/2}=L$$ is Cholesky decomposition of $$\Sigma^{-1}$$)

$$=(L^T.(X-\mu))^T.(L^T.(X-\mu))$$
Now, $$L^T.(X-\mu)$$ is projection of $$X-\mu$$ onto $$L$$, which is similar to our familiar projection in eigenspace of $$\Sigma$$, but also normalized by $$\sqrt{eigenvalue}$$'s as I show below.

Since $$\Sigma$$ is a symmetric matrix (and hopefully positive definite matrix),
$$\Sigma = A.\Lambda.A^T$$ .. (can be done via SVD decomposition)
where,
A: Matrix of eigenvectors, such that $$A^{-1} = A^T$$
$$\Lambda$$: Diagonal Matrix of eigenvalues.

Since $$\Lambda$$ is diagonal matrix (all positive values as we assumed $$\Sigma$$ is positive definite), we can simply take square root of its elements and write it as,
$$\Lambda = (\Lambda^{1/2}).(\Lambda^{1/2})^T$$

Now,
$$\Sigma = A.\Lambda.A^T = (A.\Lambda^{1/2}).(A.\Lambda^{1/2})^T$$
$$\rightarrow \Sigma^{-1} = ((\Lambda^{1/2})^{-1}.A^{-1})^T.((\Lambda^{1/2})^{-1}.A^{-1})$$
$$\rightarrow \Sigma^{-1} = (\Lambda^{-1/2}.A^T)^T.(\Lambda^{-1/2}.A^T) = L.L^T$$
$$\rightarrow L^T = \Lambda^{-1/2}.A^T$$ ... this basically mean multiply each eigenvector (row of $$A^T$$) with its $$\frac{1}{\sqrt{eigenvalue}}$$, exactly what we wanted.

Hence,
we were earlier just doing (after taking top 'k' eigenvectors)
$$A^T.(X-\mu)$$ ... i.e. simply projecting $$(X-\mu)$$ onto eigenvectors.
But we should do,
$$\Lambda^{-1/2}.A^T.(X-\mu)$$ ... i.e. projecting $$(X-\mu)$$ onto eigenvectors and normalizing by $$\Lambda^{1/2}$$

Q.E.D
