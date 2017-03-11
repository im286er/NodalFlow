# NodalFlow

[![Build Status](https://travis-ci.org/fab2s/NodalFlow.svg?branch=master)](https://travis-ci.org/fab2s/NodalFlow) [![License](https://poser.pugx.org/fab2s/nodalflow/license)](https://packagist.org/packages/fab2s/nodalflow)

NodalFlow is a generic Workflow that can execute chained tasks. It is designed around simple interfaces that specifies a flow composed of executable nodes and flows. Nodes can be executed or traversed. They accept a single parameter as argument and can be set to pass their result as an argument for the next node.
Flows also accept one argument and may be set to pass their result to be used as an argument for the next node.

NodalFlow aims at organizing and simplifying data processing workflows where data may come from various generators, pass through several data processors and / or end up in various places. It makes it possible to dynamically configure and execute complex scenario in a repeatable manner. And even more important, to write Nodes that will be reusable in any workflow.

NodalFlow enforces minimalistic requirements upon nodes. This means that in most cases, you should extend `FlowAbstract` to implement the required constraints for your use case.

[YaEtl](https://github.com/fab2s/YaEtl) is an example of a more specified workflow build upon [NodalFlow](https://github.com/fab2s/NodalFlow).

NodalFlow shares conceptual similarities with [Transduction](https://en.wikipedia.org/wiki/Transduction) as it allow basic interaction chaining, but the comparison diverges quickly.


## NodalFlow Citizens

A Flow is an executable workflow composed of a set of executable Nodes. They all carry a somehow executable payload carrying the execution logic and can be of three kinds :

* Exec Nodes :

    An Exec Node is a Node that exposes an exec method accepting one parameter and eventually returning one value that may or may not be used as argument to the next node in the flow.

* Traversable Node :

    A Traversable Node is a not that exposes a getTraversable method one parameter and returns with a Traversable which may or may not spit values that may or may not be used as argument to the next node in the flow

* Branch Node

    Branch Node is a special case where the Node's payload is a Flow. It will be treated as an exec node which may return a value that may (which results in executing the branch within the parent's Flow's flow, as if it was part of it) or may not (which result in a true branch which will only start from a specific location in the parent's Flow'w flow) be used as argument to the next node in the flow.
    Branch Nodes cannot be traversed. It is not a technical limitation, but rather something that requires further thinking and may be later implemented.

Each Node share the same constructor signature :

```php
    /**
     * As a node is supposed to be immutable, and thus
     * have no setters on $isAReturningVal and $isATraversable
     * we enforce the constructor's signature in this interface
     * One can of course still add defaulting param in extend
     *
     * @param mixed $payload
     * @param bool  $isAReturningVal
     * @param bool  $isATraversable
     *
     * @throws \Exception
     */
    public function __construct($payload, $isAReturningVal, $isATraversable = false);
```

The current version comes with three instanciable Nodes :

* CallableNode

    For conviniency, `CallableNode` implemments both `ExecNodeInterface` and `TraversableNodeInterface`. It's thus up to you to use a suitable Callable for each case.
    ```php
    use fab2s\NodalFlow\Nodes\CallableNode;

    $callableExecNode = new CallableNode(function($param) {
        return $param + 1;
    }, true);

    // which allows us to call the closure using
    $result = $callableExecNode->exec($param);

    $callableTraversableNode = new CallableNode(function($param) {
        for($i = 1; $i < 1024; $i++) {
            return $param + $i;
        }
    }, true, true);

    // which allows us to call the closure using
    foreach ($callableTraversableNode as $result) {
        // execute next nodes
    }
    ```

* BranchNode

    ```php
    use fab2s\NodalFlow\Nodes\BranchNode;

    $rootFlow = new ClassImplementingFlwoInterface;

    $branchFlow = new ClassImplementingFlwoInterface;
    // feed the flow
    // ...

    $rootFlow->addNode(new BranchNode($flow, false));
    ```

* ClosureNode

    ClosureNode is not bringing anything really other than providing with another example. It is very similar to CallableNode except it will only accept Closure as payload.

NodalFlow also comes with a NodeFactory to ease Node instanciation :
```php
use fab2s\NodalFlow\NodeFactory;

$node = new NodeFactory(function($param) {
    return $param + 1;
}, true);

$node = new NodeFactory('trim', true);

$node = new NodeFactory([$someObject, 'someMethod'], true);

$node = new NodeFactory('SomeClass::someTraversableMethod', true, true);


$branchFlow = new ClassImplementingFlwoInterface;
// feed the flow
// ...

$node = new NodeFactory($branchFlow, true);

// ..
```

And one instanciable Flow :

* CallableFlow

    ```php
    use fab2s\NodalFlow\CallableFlow;
    use fab2s\NodalFlow\NodeFactory;

    $branchFlow = new ClassImplementingFlwoInterface;
    // feed the branch flow
    // ...

    $callableFlow = new CallableFlow;
    $result = $callableFlow->add('trim', true)
        ->add(('SomeClass::someTraversableMethod', true, true))
        ->add(function($param) {
            return $param + 1;
        }, true)
        ->add(function($param) {
            for($i = 1; $i < 1024; $i++) {
                return $param + $i;
            }
        }, true, true)
        ->add($branchFlow, false)
        ->add([$someObject, 'someMethod'], false)
        ->exec();
    ```

As you can see, it is possible to dynamically generate and organize tasks which may or may not be linked together by their argument and return values

## Serialization

As the workflow became an object, it became serializable, but this is unless it carries Closures. Closure serialization is not natively supported by PHP, but there are ways around it like [Opis Closure](https://github.com/opis/closure)


## Requirements

NodalFlow is tested against php 5.6, 7.0, 7.1 and hhvm, but it may run bellow that (might up to 5.3).


## License

NodalFlow is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).