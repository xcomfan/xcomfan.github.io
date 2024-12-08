# Practical Tips on Postgres

* Normalize your data unless you have a good reason not to
* Follow all the advice from the folks that make Postgres
- ﻿﻿Note some general SQL eccentricities
- ﻿﻿Saving your pinkies: you don't have to write SQL in all caps  
    NULL is weird
- ﻿﻿You can make psql more useful
- ﻿﻿Fix your unreadable output
- ﻿﻿Clarify ambiguous nulls
- ﻿﻿Use auto-completion
- ﻿﻿Lean on backslash shortcuts
- ﻿﻿Copy to a CSV
- ﻿﻿Use column shorthands and aliases
- ﻿﻿It's possible that adding an index will do nothing (particularly if it's misconfigured)
- ﻿﻿What is an index?
- ﻿﻿An index isn't much use for a table with barely any rows in it
- ﻿﻿When indexing multiple columns, the order matters
- ﻿﻿If doing prefix matches, use text_ pattern_ops
- ﻿﻿Long-held locks can break your app (even ACCESS SHARE )
- ﻿﻿What is a lock?
- ﻿﻿How locks work in Postgres
- ﻿﻿How this can cause problems
- ﻿﻿Long-running transactions can be just as bad
- ﻿﻿JSONB is a sharp knife
- ﻿﻿JSONB can be slower than normal columns
- ﻿﻿JSONB is not as self-documenting as a standard table schema
- ﻿﻿JSONB Postgres types are a bit awkward to work wit

