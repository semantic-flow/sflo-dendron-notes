---
id: WOX0nOdb7lQ4qrz2VC1wh
title: Project Bia
desc: ''
updated: 1729355786291
created: 1632928184201
---
  
potency/effectiveness through low-friction, well-organized, prioritized task and goal management ^nY8csQYnizlE

## Inspiration

- "an aliveness-based system works better than an exhaustiveness-based system for people who are pursuing purposeful goals" 

## Conventions

- put a tag after the square bracket box, so you can collect all the task from everywhere with a find
  - could even use multiple tags and regex searches
- when you come back to something, tag with a date.
  - the info is recoverable from source, but that's not convenient
    - maybe gitlense

## [[Task Management|t.tm]]

- tasks can live in any single place; tasks can be multi-homed with references/transclusions

## User Stories

- I am taking notes of every discussion and meeting I attend using markdown. Most of the things I need to do are identified in these notes. Transferring those tasks from the notes to another tool is not only cumbersome and risky, I lose a lot of the context in the process. I came to realize that in practice, the divide between notes and tasks is not clear.[^1] ^rxGIZrgVBtUO

### Activity Statuses

- [` `] : to-do (middle-priority)
- [`-`] : to-do backlog / low-priority
- [`+`] : to-do high-priority
- [`!`] : to-do urgent
- [`x`] or [ t.2023.11.27 ] or [ t.2023.11.27.10.09 ]: done (or done on date)
- [`c`] : cancelled
- [`?`] : waiting for feedback / blocked
- [`>] : duplicatedTo/spill-over; or moved to another note or another system (Jira, `proj.` hierarchy, sunsama )
  - nice to link a trace
- resources:  ["checkbox states" from Dendron docs](https://wiki.dendron.so/notes/593206ea-5658-4874-bafd-18a138870f91boo/)


### activity predicates

- [[p.did]] (implied if no other predicate explicitly stated)
- [[p.read]]
- [[p.watched]]
- [[p.listened]] to/for/at/with
- [[p.administrated]]
 

### tasks vs activities (posible-future-activities)

- activities can be bigger than tasks, i.e., a whole note to themselves, but still can represent the activity as a whole completion status with the task status

### tasks vs projects/goals

- tasks (e.g. `- [] items` scattered through the topics, addressable-resources, and user hierarchies) don't have "start by" or "finish by" timing attached
- due date can be distinct from finish by, but pragmatically they usually align
- tasks become projects/goals when they're added to a daily journal, event page, 
- project feels bigger, but think "micro-projects are projects too: tasks with planned start and/or finishes"
- [[formulate your Project List and then replicate that list across every single tool you use, now and in the future.|t.km.PARA#^IoqWYbtXfBja]]

### tasks vs plans

- plans are elaborated (or reflected-upon) tasks

### tasks vs vision

- put simply, vision is what you want to see
  - vision is a little fuzzy
  - can have a goal attached, but goal usually develops as time spent understanding priorities, specifying success criteria,  and resource constraints increases
## Goals / Project Management

- [ ] consider a "goal" hierarchy
  - complements or supported by (or integrated with?) project hierarchy
  - maybe also
    - dream: for goals without projects
    - hobby: for projects without goals 
- `p.hadGoal` should probably reference something in Jira, Asana, or ??
  - Jira: defintiely for coding projects or collaboration (hopefully someday soon)
- ultimately, Dendron notes are not sufficient
- maybe a separate calendar is the way to go for goals

### goals vs objectives

- ["Goals are the distinct purpose that is to be anticipated from the assignment or project,[3] while objectives, on the other hand, are the determined steps that will direct full completion of the project goals."](https://en.wikipedia.org/wiki/SMART_criteria#cite_note-:0-3)


## Daily Routine

- brain dump 
  - 2021-11-16 this feels like a reference to something, but can't recall. 
- maybe not absolutely necessary to list already-scheduled items, but good idea to review schedule
  - since you'll be recording those things anyhow, might as well get them inputted 
  - but do I really differentiate between planned and did?
- copy ~~tasks~~ references into your daily journal note under `p.sawPlanning`
  - look for high-priority
  - reference weekly goals
- GTD
- maybe [[p.sawUnplannedActivity]]
- mark planned items as complete or cancelled or pushed

## Weekly routine

- review previous daily journal notes and previous weekly goals
- review big picture
- set some new goals
- #optional do some task grooming and project planning
- #optional update [[now]]
- #optional update [[big-picture]]

## Monthly/Quarterly/Annually

- review [[big-picture]]
- set some new goals
- #optional update [[big-picture]]
  
## Resources

- [^1]: https://feval.ca/posts/pw/ 
