# Spread the virus

## Task
1. One node of the graph is _infected_ and identified through a 1 in one element of its feature vector. 
   The remaining features are unused. The infection is spread to the neighbors (directed) of the infected nodes.
   
2. One node of the graph is _infected_ and identified through a 1 in the first element of its feature vector. Some nodes
   are immune and identified through a 1 in the second element of their feature vectors. 
   Again, the infection is spread to the neighbors, but some of them are immune.
   
The network should output a prediction of 1 for nodes that are infected and 0 for the others, effectively
identifying the neighbors of the infected node, the connection type and the immunity status. 

## Generalization strategy

## Weighting function

## Full network
The full version of the graph network operates as such:
```
e_{s \to r}^{t+1} = ReLU [ f_e(e_{s \to r}^t) + f_s(n_s^t) + f_r(n_r^t) + f_u(u^t)]

n_i^{t+1} = Sigmoid [ g_n(n_i^t) + g_{in}(agg_s(e_{s \to i}^{t+1})) + g_{out}(agg_r(e_{i \to r}^{t+1})) + g_u(u^t)]

u_i^{t+1} = h_n(agg_i(n_i^{t+1})) + h_e(agg_{ij}(e_{i \to j}^{t+1})) + h_u(u^t)
```

using max as an aggregation function

## Minimal network
For this task, the minimal version of the network should only use:

## Workflow

1. Create base folder
    ```bash
    INFECTION=~/experiments/infection/
    mkdir -p "$INFECTION/{runs,data}"
    ```
2. Create dataset
    ```bash
    python -m infection.dataset generate <dataset.yml> folder="$INFECTION/data"
    ```
3. Create tensorboard layout
    ```bash
    python -m infection.layout --folder "$INFECTION/runs"
    ```
    
4. Start a mongo db in a docker container and note its ip address
    ```bash
    docker pull mongo:latest
    docker run -d --name=mongo mongo
    docker inspect mongo | grep -w IPAddress
    docker network inspect bridge
    ```
   
5. Launch experiments
    ```bash
    python -m infection.train -m <dbaddr:dbport>:sacred with <model.yaml> <train.yaml> [<other config>]

    conda activate tg-experiments
    for i in 1 2 3 4 5; do
       for type in full minimal; do
           for lr in .01 .001; do
               for wd in 0 .01 .001; do
                   python -m infection.train \
                       -m 172.17.0.2:27017:sacred \
                       with ../config/infection/{train,${type}}.yaml \
                       paths.root=~/experiments/infection \
                       opts.session=${type}_lr${lr}_wd${wd}_${i} \
                       optimizer.lr=${lr} \
                       training.l1=${wd} \
                       training.epochs=20 \
                       training.batch_size=1000
               done
           done
       done
    done
    ```
6. Query logs and visualize
   - Tensorboard: `tensorboard --logdir "$INFECTION/runs"`
   - Mongo shell `docker run -it --rm mongo mongo <dbaddr:dbport>/sacred`
   - Omniboard `docker run -it --rm -p 9000:9000 --name omniboard vivekratnavel/omniboard -m 172.17.0.2:27017:sacred`