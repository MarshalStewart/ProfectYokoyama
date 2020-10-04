
# Training Interface

Tensorpack trainers have an interface for maximum flexibility.
Then, there are interfaces built on top of trainers to simplify the use,
when you don't want to customize too much.

### Raw Trainer Interface

__Define__: For general trainer, build the graph by yourself.
For single-cost trainer, build the graph by
[SingleCostTrainer.setup_graph](../modules/train.html#tensorpack.train.SingleCostTrainer.setup_graph).

__Run__: Then, call
[Trainer.train()](../modules/train.html#tensorpack.train.Trainer.train)
or
[Trainer.train_with_defaults()](../modules/train.html#tensorpack.train.Trainer.train_with_defaults)
which applies some defaults options for normal use cases.

### With ModelDesc and TrainConfig

This is an interface that's most familiar to old tensorpack users,
and is now mainly useful for single-cost tasks.
A lot of examples are written in this interface.

[SingleCost trainers](trainer.html#single-cost-trainers)
expects 4 arguments in `setup_graph`: `InputDesc`, `InputSource`, get_cost function, and an optimizer.
`ModelDesc` describes a model by packing 3 of them together into one object:

```python
class MyModel(ModelDesc):
	def _get_inputs(self):
		return [InputDesc(...), InputDesc(...)]

	def _build_graph(self, inputs):
		tensorA, tensorB = inputs
		# build the graph
		self.cost = xxx	 # define the cost tensor

	def _get_optimizer(self):
	  return tf.train.GradientDescentOptimizer(0.1)
```

`_get_inputs` should define the metainfo of all the inputs your graph will take to build.

`_build_graph` takes a list of `inputs` tensors which will match `_get_inputs`.

You can use any symbolic functions in `_build_graph`, including TensorFlow core library
functions and other symbolic libraries.
But you need to follow the requirement of
[get_cost_fn](../modules/train.html#tensorpack.train.SingleCostTrainer.setup_graph),
because this function will be used as part of `get_cost_fn`.
At last you need to set `self.cost`.

After defining such a model, use it with `TrainConfig` and `launch_train_with_config`:

```python
config = TrainConfig(
   model=MyModel()
   dataflow=my_dataflow,
   # data=my_inputsource, # alternatively, use a customized InputSource
   callbacks=[...],		# some default callbacks are automatically applied
	 # some default monitors are automatically applied
	 steps_per_epoch=300,	 # default to the size of your InputSource/DataFlow
)

trainer = SomeTrainer()
# trainer = SyncMultiGPUTrainerParameterServer([0, 1, 2])
launch_train_with_config(config, trainer)
```
See the docs of
[TrainConfig](../modules/train.html#tensorpack.train.TrainConfig)
and
[launch_train_with_config](../modules/train.html#tensorpack.train.launch_train_with_config)
for usage and detailed functionalities.
