# Foreword

> 2022.01.21

- small things matter
- 80% or more of what we do is "maintenance"
- consistent indentation style was one of the most significant indicator of low bug report
- the design lives in the code
- design as a process not a static endpoint.
- Did we do our best to leave the campground cleaner than we found it?
- Neither architecture nor clean code insist on perfection, only honesty and doing the best we can

# Chapter 1. Clean Code

> 2022.01.23

- Later equals never
- We should not be shy about telling non-developers/coworkers what we think
- The only way to make the deadline - the only way to go fast - is to keep the code as clean as possible at all times
- good code: Good for reading, efficient, easy for testing, ...
- no rules but suggetions of good programmers
- The Boy Scout Rule: Leave the campground cleaner than you found it

# Chapter 2. Meaningful Names

> 2022.01.27

- If a name requires a comment, then the name does not reveal its intent
- Distinguish names in such a way that the reader knows what the differneces offer. \
  no account, accountData, customer, customerInfo!
- Use searchable names
- avoid mental mapping
- One difference between a smart programmer and the professional programmer is that the professional understands the clarity is king.
- Pick one word per concept

# Chapter 3. Functions

> 2022.01.28

- Small!!!
- Do one thing
- We want the code to read like a top-down narrative
- The less input arguments are the better. \
  If more than 2, consider using class to wrap them
- no side effects
- Seperate the command from the query. \
  Don't set and use if in the same line
- Prefer exceptions to returnning error codes
- Function should do one thing and error handling is one thing
- Don't repeat yourself
- The first draft might be clumsy, but keep refine it until it reads the way you want it to read.
- Master programmers think of systems as stories to be told rather than programs to be written.

# Chapter 4. Comments

> 2022.01.28

- The best is no need to comment.
- Comment is a failure. The preoper use of commnets is to compensate for our failure to express ouself in code
- Codes change but comments don't always follow them.
- Comments do not make up for bad code
- Explain yourself in code.
- Good comments
  - Informative
  - Explanation of intent
  - Warning of consequence
  - TODO
- Bad comments
  - Redundant, noise comments(unneeded)
  - Attributions, bylines(author)
  - Nonlocal information: the comment should describe the code it appears near. No system wide information
