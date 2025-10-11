# PLANNING

## CREATING A PLAN

When we are working on a plan, I want you to be exhaustive; ask clarifying questions of the operator. Be thorough. Once you have enough information to create your plan, you must do the following.

You must break your plan up into large groups (called phases), these are ordered, and they must have subtasks in them that are reasonably sized for one person or LLM coding agent to tackle at a time. Each subtask should have examples if possible or at least an explicit definition of done; the overall phase must also have both examples and a definition of done. Code examples are also a good idea!

### PERSISTING THE PLAN

Once you have decomposed the plan into phases with subtasks, you must write out a plan folder in the ./docs/plans folder, with the followin structure:

`docs/plans/{plan_name}/overview.md`: a file that contains a high level explanation of the work, pitched so a non-technical coworker can read it and understand. Each _phase_ of the plan must have a `State:` line in the description that has one of the following _todo_, _in-progress_, _completed_, or _failed_ that represents the stte of the work.

`docs/plans/{plan_name}/phase_{n}.md` : for each phase, the details, including code examples, rationale, plan, and subtasks goes in here. This should be *EXHAUSTIVE*. Each subtask must have a `[ ] competed` checkbox; this is how we will track our progress through the plan. 

### CREATING TRACKING ISSUES

Then, you must ask the operator for the Github Project that is associated with this task; you then must create new issues, one per phase, with the subtasks as a checklist in that project. You must assign them to the user the operator provides. You must tag these issues with the tags that the operator provides, as well as other tags that allow us to group work ("database", "testing", &c.)

Once you have created these issues, you *MUST* update the plan documents (both the overview, and the specific phase files) with URLs to them, so we can use the plan document as the canonical source of truth.

### CREATING ASANA TICKET FOR VISIBILITY

Finally, the operator will tell you which **ASANA** project this plan document is to live in; and you will create an Asana ticket in that project with the contents of the overview file; and links to the github issues per phase. You will then also update the overview with a link to this Asana ticket.

## EXECUTING ON A PLAN

When you are asked to iterate over a plan, I want you to use this workflow.

### PRE-REQUISITES

You are to start from a clean git status of the current repository. If there are changes, you *MUST* *refuse to continue, and escalate to the operator.

Then, you are to read the overview for the plan; restate the overall goal, read the current state of the Github issues. If there is variance between the `State:` headers in the overview and the state of the `[ ]` completed check marks in the phase documents, you *MUST* stop and have the operator instruct you on how to continue.

### PHASES

Starting with the first incomplete phase in the overview, create a local git branch, called `{plan_name}_phase_{n}` where `n` is the phase number; you will do all your work in this branch. If this branch already exists, then you are to *warn the operator* and you *MUST* stop at that point.

Assuming you have switched to this phase branch, you are to work sequentially on the subtasks in the `docs/plans/{plan_name}/phase_{n}.md` file; as you complete each subtask, you are to create a Conventional Commit, and update the version number accordingly (`versioneer patch` for patch level changes; `versioneer minor` for minor level ones, and `versioneer major` for major changes.) Subtasks will almost certainly be `patch` changes. As you complete work, update the Github issue, the planning documents, and the Asana ticket with the git SHA hash, as well as any information we gathered in the execution stages.

When you finish a phase, you are to rewrite git history *for only the commits you've made in the phase* down to a single commit. If you continue onto a new phase, you are to create a new branch **from this phase branch**, named `{plan_name}_phase_{n + 1}`. This way, we maintain a linked list, and can return to an earlier phase if necessary in a clean fashion.

If the operator tells you to stop AFTER A PHASE IS COMPLETE, **OR** you finish the last phase in the plan; you are to *consolidate the commits into one*, with an exhaustive commit message explaining the work, any issues we found along the way, and changes we made to the initial plan; update the documents accordingly,  and then merge this branch *back into the base branch*. This way, the repository is clean, with a single commit for the work that we did together.

If you resume work on a `{plan_name}_phase_{n}` branch, you *MUST* research the current state of the code, the Github issues, *and the planning documents*. If there are discrepancies, you are to **STOP IMMEDIATELY** and **ALERT THE OPERATOR**, and **WAIT FOR GUIDANCE**.

## COMPLETING A PLAN

Once we have completed all the work in our plan, you must *MARK THE PLAN AS COMPLETED* in teh Asana ticket, with a brief (1 paragraph max) explanation of the work, where the work diverged from the assumptions in the original ticket. At this point, you *MUST* move the plan directory from `docs/plans/{plan_name}` to `docs/completed/{plan_name}`, and update the `docs/map.md` document map file.


