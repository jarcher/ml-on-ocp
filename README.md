# This repo's content is for building and deploying CUDA/GPU-enabled ML/AI images on Openshift 3.6.
## You must first build the base image (adds cuda layer)
## You can create your own Dockerfile for whatever your ML platform is.  This one is for Tensorflow+Jupyter

<code>
# Env vars
NODE_NAME=hv4.home.nicknach.net\n
GPU_NAME=GTX_970\n
PROJECT=ml-on-ocp\n
GIT=https://github.com/nnachefski/ml-on-ocp.git\n
APP=jupyter\n
</code>

1.  Join a bare-metal Openshift 3.6 node to your cluster w/ a CUDA-enabled NVIDIA GPU (label that node appropriately)
	`oc label node $NODE_NAME alpha.kubernetes.io/nvidia-gpu-name='$GPU_NAME' --overwrite`

2.  create the project
	`oc new-project ml-on-ocp`

3.  set anyuid for the default serviceaccount
	oc adm policy add-scc-to-user anyuid -z default

4.  set an alias to refresh from github
<code>
	alias refdemo='cd ~; rm -rf $PROJECT; git clone $GIT; cd $PROJECT'
</code>

5.  now do the refresh
<code>
	refdemo
</code>

6.  now build the base image
<code>
	oc new-build . --name=rhel7-cuda --context-dir=rhel7-cuda
</code>

7.  now build/deploy the AI/ML framework
<code>
	oc new-app . --name=jupyter
</code>

8.  expose the jupyter UI port
<code>
	oc expose svc jupyter --port 8888
</code>

9.  test the mnist notebook, it will run on the general CPU (use top), then patch the dc to set resource limits and nodeaffinity 
<code>
	oc patch dc $APP -p '{"spec":{"template":{"spec":{"affinity":{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"alpha.kubernetes.io/nvidia-gpu-name","operator":"In","values":["GTX_970"]}]}]}}},"containers":[{"name":"$APP","resources":{"limits":{"alpha.kubernetes.io/nvidia-gpu":"1"}}}]}}}}'
</code>

## now run the mnist notebook again and see that it scheduled on the GPU (use nvidia-smi on the bare-metal node)

