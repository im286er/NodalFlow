# Exceptions

When an `Exception` is thrown during execution, NodalFlow catches it, perform few cleanup operations, including triggering the proper callback and then re-throws it as is if it's a `NodalFlowException` or else throws a `NodalFlowException` with the original exception set as previous. This means that NodalFlow will only throw `NodalFlowException` unless an exception is raised within an exception callback of yours.

`NodalFlowException` extends [ContextException](https://github.com/fab2s/ContextException) formalizing a common context exception pattern (as in Symfony for example) to ease `Exception` logging.

You can thus add context to exception when implementing nodes which will make it easy to later retrieve and log (MonoLog/sentry etc ...).
