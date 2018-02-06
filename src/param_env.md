# Parameter Environment

When working with associated constants or generic types it is often relevant to
have more information about the `Self` or generic parameters. Trait bounds and
similar information is encoded in the `ParamEnv`. Often this is not enough
information to obtain things like the type's `Layout`, but you can do all kinds
of other checks on it (e.g. whether a type implements `Copy`) or you can
evaluate an associated constant whose value does not depend on anything from the
parameter environment.

Although you can obtain a valid `ParamEnv` for any item via
`tcx.param_env(def_id)`, this `ParamEnv` can be too generic for your use case.
Using the `ParamEnv` from the surrounding context can allow you to evaluate more
things.

Another great thing about `ParamEnv` is that you can use it to eliminate the
generic parameters from a `Ty` by calling `param_env.and(ty)`.
