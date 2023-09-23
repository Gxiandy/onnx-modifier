<img src="./docs/onnx_modifier_logo.png" style="zoom: 60%;" />

English | [简体中文](README_zh-CN.md)

# Introduction

To edit an ONNX model, one common way is to visualize the model graph, and edit it using ONNX Python API. This works fine. However, we have to code to edit, then visualize to check. The two processes may iterate for many times, which is time-consuming. 👋

What if we have a tool, which allows us to **edit and preview the editing effect in a totally visualization fashion**?

Then `onnx-modifier` comes. With it, we can focus on editing the model graph in the visualization pannel. All the editing information will be summarized and processed by Python ONNX API automatically at last. Then our time can be saved! 🚀

`onnx-modifier` is built based on the popular network viewer [Netron](https://github.com/lutzroeder/netron) and the lightweight web application framework [Flask](https://github.com/pallets/flask).

Currently, the following editing operations are supported:

:white_check_mark: [Delete/recover nodes](#Delete_or_recover_nodes)<br>
:white_check_mark: [Add new nodes](#Add_new_nodes)<br>
:white_check_mark: [Rename the node inputs/outputs](#Rename_the_node_inputs_and_outputs)<br>
:white_check_mark: [Rename the model inputs/outputs](#Rename_the_model_inputs_and_outputs)<br>
:white_check_mark: [Add new model outputs](#Add_new_model_outputs)<br>
:white_check_mark: [Edit attribute of nodes](#Edit_attribute_of_nodes)<br>
:white_check_mark: [Edit batch size](#Edit_batch_size)<br>
:white_check_mark: [Edit model initializers](#Edit_model_initializers)<br>

Here is the [update log](./docs/update_log.md) and [TODO list](./docs/todo_list.md).

Hope it helps!

# Getting started

We have three methods to launch `onnx-modifier` now.

## launch from command line
Clone the repo and install the required Python packages by

```bash
git clone git@github.com:ZhangGe6/onnx-modifier.git
cd onnx-modifier

pip install -r requirements.txt
```

Then run

```bash
python app.py
```

Click the url in the output info generated by flask (defaults to `http://127.0.0.1:5000/`), then `onnx-modifier` will be launched in the web browser.

## launch from executable file
<details>
  <summary>Click to expand</summary>

- Windows: Download onnx-modifier.exe (27.6MB) [Google Drive](https://drive.google.com/file/d/1LRXgZauQ5BUENe_PvilRW8WvSO-4Jr9j/view?usp=sharing) / [Baidu NetDisk](https://pan.baidu.com/s/1pc6rR6bt29y2ewl_6uapHQ?pwd=vh32), double-click it and enjoy.
  - Edge browser is used for runtime environment by default.

> I recorded how I made the the executable file in `app_desktop.py`. The executable file for other platforms are left for future work.

</details>



## launch from a docker container
<details>
  <summary>Click to expand</summary>

We create a docker container like this:

```bash
git clone git@github.com:ZhangGe6/onnx-modifier.git
cd onnx-modifier
docker build --file Dockerfile . -t onnx-modifier
```

After building the container, we run onnx-modifier by mapping docker port and a local folder `modified_onnx`

```bash
mkdir -p modified_onnx
docker run -d -t \
  --name onnx-modifier \
  -u $(id -u ${USER}):$(id -g ${USER}) \
  -v $(pwd)/modified_onnx:/modified_onnx \
  -p 5000:5000 \
  onnx-modifier
```

Then we have access to onnx-modifer from URL <http://127.0.0.1:5000>. The modified ONNX models are expected to be found inside the local folder `modified_onnx`.
</details>



Click `Open Model...` to upload the ONNX model to edit. The model will be parsed and shown on the page.

# Usage

Graph-level-operation elements are placed on the left-top of the page. Currently, there are three buttons:  `Reset`, `Download` and `Add node`. They can do:
- `Reset`: Reset the whole model graph to its initial state;
- `Download`: Save the modified model into disk. Note the two checkboxes on the right
  - (**experimental**) select `shape inferece` to do [shape inferece](https://github.com/onnx/onnx/blob/main/docs/ShapeInference.md) when saving model.
    - The `shape inferece` feature is built on [onnx-tool](https://github.com/ThanatosShinji/onnx-tool), which is a powerful ONNX third-party tool.
  - (**experimental**)  select `clean up` to remove the unused nodes and tensors (like [ONNX GraphSurgeon](https://docs.nvidia.com/deeplearning/tensorrt/onnx-graphsurgeon/docs/ir/graph.html#onnx_graphsurgeon.Graph.cleanup)).
- `Add node`: Add a new node into the model.

Node-level-operation elements are all in the sidebar, which can be invoked by clicking a specific node.

Let's take a closer look.

## Delete/recover nodes<a id='Delete_or_recover_nodes'></a>
There are two modes for deleting node: `Delete With Children` and `Delete Single Node`. `Delete Single Node` only deletes the clicked node, while `Delete With Children` also deletes all the node rooted on the clicked node, which is convenient and natural if we want to delete a long path of nodes.

> The implementation of `Delete With Children` is based on the backtracking algorithm.

For previewing, The deleted nodes are in grey mode at first. If a node is deleted by mistake, `Recover Node` button can help us recover it back to graph. Click `Enter` button to take the deleting operation into effect, then the updated graph will show on the page automatically.

The following figure shows a typical deleting process:

<img src="./docs/delete_node.gif" style="zoom:75%;" />

## Add new nodes<a id='Add_new_nodes'></a>
Sometimes we want to add new nodes into the existed model. `onnx-modifier` supports this feature experimentally now.

Note there is an `Add node` button, following with a selector elements on the top-left of the index page. To do this, what we need to do is as easy as 3 steps:

1. Choose a node type in the selector, and click `Add node` button. Then an empty node of the chosen type will emerge on the graph.

   > The selector contains all the supported operator types in domains of `ai.onnx`(171), `ai.onnx.preview.training`(4), `ai.onnx.ml`(18) and `com.microsoft`(1).

2. Click the new node and edit it in the invoked siderbar. What we need to fill are the node Attributes (`undefined` by default) and its Inputs/Outputs (which decide where the node will be inserted in the graph).

3. We are done.

<img src="./docs/add_new_node.gif" style="zoom:75%;" />

The following are some notes for this feature:

1. By clicking the `?` in the `NODE PROPERTIES -> type` element, or the `+` in each `Attribute` element, we can get some reference to help us fill the node information.

2. It is suggested to fill all of the `Attribute`, without leaving them as `undefined`.  The default value may not be supported well in the current version.

3. For the `Attribute` with type `list`, items are split with '`,`' (comma). Note that `[]` is not needed.

4. For the `Inputs/Outputs` with type `list`, it is forced to be at most 8 elements in the current version. If the actual inputs/outputs number is less than 8, we can leave the unused items with the name starting with `list_custom`, and they will be automatically omitted.

## Rename the name of node inputs/outputs<a id='Rename_the_node_inputs_and_outputs'></a>

By changing the input/output name of nodes, we can change the model forward path. It can also be helpful if we want to rename the model output(s).

Using `onnx-modifier`, we can achieve this by simply enter a new name for node inputs/outputs in its corresponding input placeholder. The graph topology is updated automatically and instantly, according to the new names.

For example,  Now we want remove the preprocess operators (`Sub->Mul->Sub->Transpose`) shown in the following figure. We can

1. Click on the 1st `Conv` node, rename its input (X) as *serving_default_input:0* (the output of node `data_0`).
2. The model graph is updated automatically and we can see the input node links to the 1st `Conv`directly. In addition, the preprocess operators have been split from the main routine. Delete them.
3. We are done! (click `Download`, then we can get the modified ONNX model).

> Note: To link node $A$ (`data_0` in the above example) to node $B$ (the 1st `Conv` in the above example), **it is suggested to edit the input of node $B$ to the output of node `A`, rather than edit the output of node $A$ to the input of node `B`.** Because the input of $B$ can also be other node's output (`Transpose`  in the above example ) and unexpected result will happen.

The process is shown in the following figure:

<img src="./docs/rename_io.gif" style="zoom:75%;" />

## Rename the model inputs/outputs<a id='Rename_the_model_inputs_and_outputs'></a>
Click the model input/output node, type a new name in the sidebar, then we are done.

![rename_model_io](./docs/rename_model_io.gif)

## Add new model outputs<a id='Add_new_model_outputs'></a>

Sometimes we want to add/extract the output of a certain node as model output. For example, we want to add a new model output after the old one was deleted, or extract intermediate layer output for fine-grained analysis. In `onnx-modifier`, we can achieve this by simply clicking the `Add Output` button in the sidebar of the corresponding node. Then we can get a new model output node following the corresponding node. Its name is the same as the output of the corresponding node.  

In the following example, we add 2 new model outputs, which are the outputs of the 1st `Conv` node and 2nd `Conv` node, respectively.

![add_new_outputs](./docs/add_new_outputs.gif)

## Edit attribute of nodes<a id='Edit_attribute_of_nodes'></a>

Change the original attribute to a new value, then we are done.

> By clicking the `+` in the right side of placeholder, we can get some helpful reference.

<img src="./docs/change_attr.gif" style="zoom:75%;" />

## Edit batch size<a id='Edit_batch_size'></a>
`onnx-modifier` supports editing batch size now. Both `Dynamic batch size` and `Fixed batch size` modes are supported.
- `Dynamic batch size`: Click the `Dynamic batch size` button, then we get a model which supports dynamic batch size inferece;
- `Fixed batch size`: Input the fixed batch size we want, then we are done;

<img src="./docs/rebatch.gif" style="zoom:75%;" />

Note the differences between `fixed batch size inference` and `dynamic batch size inference`, as [this blog](https://nietras.com/2021/05/24/set-dynamic-batch-size-using-onnx-sharp/) illustrates:
> - When running a model with only fixed dimensions, the ONNX Runtime will prepare and optimize the graph for execution when constructing the Inference Session.
> -  when the model has dynamic dimensions like batch size, the ONNX Runtime may instead cache optimized graphs for specific batch sizes when inputs are first encountered for that batch size.

## Edit model initializers<a id='Edit_model_initializers'></a>
Sometimes we want to edit the values which are stored in model initializers, such as the weight/bias of a convolution layer and the shape parameter of a `Reshape` node. `onnx-modifier` supports this feature now! Input a new value for the initializer in the invoked sidebar and click Download, then we are done.

<img src="./docs/edit_initializer.gif" style="zoom:75%;" />

> Note: For the newly added node, we should also input the datatype of the initializer. (If we are not sure what the datatype is, click `NODE PROPERTIES->type->?`, we may get some clues.)

# Sample models

For quick testing, some typical sample models are provided as following. Most of them are from [onnx model zoo](https://github.com/onnx/models)

- squeezeNet [Link (4.72MB)](https://github.com/onnx/models/blob/main/vision/classification/squeezenet/model/squeezenet1.0-12.onnx)
- MobileNet [Link (13.3MB)](https://github.com/onnx/models/blob/main/vision/classification/mobilenet/model/mobilenetv2-7.onnx)
- ResNet50-int8 [Link (24.6MB)](https://github.com/onnx/models/blob/main/vision/classification/resnet/model/resnet50-v1-12-int8.onnx)
- movenet-lightning [Link (9.01MB)](https://pan.baidu.com/s/1MVheshDu58o4AAgoR9awRQ?pwd=jub9)
  - Converted from the pretrained [tflite model](https://tfhub.dev/google/movenet/singlepose/lightning/4) using [tensorflow-onnx](https://github.com/onnx/tensorflow-onnx);
  - There are preprocess nodes and a big bunch of postprocessing nodes in the model.

`onnx-modifier` is under active development 🛠. Welcome to use, create issues and pull requests! 🥰

# Credits and referred materials

- [Netron](https://github.com/lutzroeder/netron)
- [Flask](https://github.com/pallets/flask)
- ONNX IR [Official doc](https://github.com/onnx/onnx/blob/main/docs/IR.md)
- ONNX Python API [Official doc](https://github.com/onnx/onnx/blob/main/docs/PythonAPIOverview.md), [Leimao&#39;s Blog](https://leimao.github.io/blog/ONNX-Python-API/)
- ONNX IO Stream [Leimao&#39;s Blog](https://leimao.github.io/blog/ONNX-IO-Stream/)
- [onnx-utils](https://github.com/saurabh-shandilya/onnx-utils)
- [sweetalert](https://github.com/t4t5/sweetalert)
- [flaskwebgui](https://github.com/ClimenteA/flaskwebgui)
- [onnx-tool](https://github.com/ThanatosShinji/onnx-tool) 👍