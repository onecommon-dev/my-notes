[Back to main page](../README.md)

# Preface

This page seeks to condense my entire expertise and background in the domain into concise, accessible principles accompanied by examples. My goal is for them to act as reliable compass points whenever you feel overwhelmed by complexity. Ultimately, mastering complexity lies at the core of software engineering.

# KISS

Before jumping into the specific principles, let me share one foundational rule. Call it a principle if you prefer: Keep It Simple, Stupid (KISS).

Good developers follow SOLID, but only when it is needed.

No serious software project survives long term without eventually respecting SOLID, yet there’s zero benefit in piling abstraction on top of abstraction if we aren’t using inheritance, or worse, if we never needed it at all. Knowing exactly when to apply these principles is an art that only experience can teach.

In the examples you’ll see here, the code often begins by breaking the rules, usually because of new or evolving requirements, and then we apply a solution to make it compliant. That messy, iterative process mirrors real world development.

Of course, seasoned developers can often anticipate upcoming needs. When experience tells you certain requirements are coming, it’s perfectly fine to adjust your modules ahead of time to stay SOLID. Doing so doesn’t violate KISS at all.

# SOLID
Go to [SOLID Principles](./SOLID.md)

# Domain Driven Design
Go to [Domain Driven Design](./DDD.md)

# Clean Architecture
Go to [Clean Architectures](./clean-arch.md)

## Law of Demeter (LoD) (Principle of least knowledge)

This video provides a good definition of LoD:
https://www.youtube.com/watch?v=L60bB4ekIpI