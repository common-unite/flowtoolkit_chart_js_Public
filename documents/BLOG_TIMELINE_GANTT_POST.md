# Gantt-Style Timelines in Flow Screens - and Click-to-Edit Records on Any Chart

This one started as a question in the Outbound Funds community: "Anyone have a Gantt chart tool they love?" A consultant's grantmaking client wanted to see Requirements - the interim and final reports every grant carries - on a timeline by due date, grouped by Funding Request, color-coded by type, filterable by tags, with details on hover. The AppExchange options were standalone apps: separate tab, separate context, no connection to the Flows the team already lived in.

Release 0.6.0.1 of Flow Tool Kit: Chart.js answers that question with two features: a **Timeline (Gantt) chart type**, and **Record Form on Click**, which lets users edit the record behind any chart element in a modal without leaving the screen.

![Timeline (Gantt) walkthrough: chained swimlanes, record editing in a modal, compact list, and grouped milestones](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/18-timeline-finalize-demo.gif)

---

## The Timeline (Gantt) Chart Type

Pick **Timeline (Gantt)** in the chart type dropdown and the property editor swaps its aggregation fields for timeline mapping: which field labels each row, which field groups rows into programs, where bars start and end, and which date renders as a milestone diamond. Like every chart type in the package, it reads any record collection your Flow already has - Requirements, Campaigns, shifts, phases, any object with dates.

One chart type covers three layouts, inferred from two fields:

- **Flat list** - a Row Label field alone gives one bar per record, in collection order.
- **Grouped swimlanes** - add a Group By field and rows organize under bold headers with alternating bands.
- **Grouped milestones** - leave Row Label blank and each group becomes a single row, its records rendered as diamonds along a neutral period bar. This is the grantmaking layout: one row per Funding Request, one diamond per Requirement due date.

![Grouped milestones with a hover tooltip](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/18-timeline-milestones-tooltip.png)

Everything renders with human labels. Lookups - including custom ones like the Funding Request lookup - resolve to record names. Picklists show translated labels. Tooltips carry any extra fields you name, description text wraps, and colors follow the same per-value mapping UI the other chart types use, so Interim reports can always be orange and Final reports always red. A dashed today line keeps everyone oriented, progress fills show completion inside each bar, and the axis ticks by week, month, quarter, or year.

And because it is a Flow screen component, the client's tag filter needed zero chart configuration: a choice picklist above the chart, a Collection Filter upstream, and the timeline re-renders reactively as the user changes the tag.

## Deadline-Only Data Still Makes a Schedule

Here is the part that surprised us during the build: real grant data often has *only due dates*. No start dates anywhere. A traditional Gantt has nothing to draw.

The timeline handles this with a fallback ladder. A record missing its start date automatically **starts where the previous record in its group ended** - collection order defines the sequence, and your upstream Sort controls collection order. Anchor each group's first record with a **Default Start Date** (bindable to a Flow variable, like the grant's award date) and deadline-only data becomes a genuine phase-by-phase schedule: enrollment runs from award to its due date, the interim report picks up from there, the final report closes out the year. Records that *do* carry real start dates always win, and the chain resumes from their end.

![Chained swimlanes with progress fill and end markers](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/18-timeline-chained-swimlanes.png)

That chaining quietly delivers the most-requested Gantt behavior - "this phase starts when the previous one ends" - without dependency arrows, drag-to-reschedule, or any of the machinery that makes PM tools heavy. It is just how the bars resolve.

## Click a Record, Edit It, Never Leave the Screen

The second feature applies to every chart type, not just timelines. Turn on **Record Form on Click** and any chart element that represents exactly one record - a milestone diamond, a timeline bar, a detail-mode bar or slice - opens that record in a FlowToolKit Form modal when clicked.

![Editing a phase in the record form modal](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/18-record-form-modal.png)

Three things make this more interesting than a quick-action popup:

**Your Flow controls the save.** The component performs no DML. Saving the modal updates the chart immediately - the bar moves, the color changes - and the edited records flow out through `updateCollection` and `hasRecordsForUpdate`, the same contract the FlowToolKit Form Table and Repeater use. Your Flow decides when and how to write, with all your validation and logic in the path.

**Every record can open its own form.** The form source follows the flowForm pattern: pick one Form for all records, build an inline custom form, or - the interesting one - read the form name from a *field on the clicked record*. A formula field can route budget Requirements to a budget form and narrative Requirements to a narrative form, from one chart, with zero Flow logic.

**Edits survive navigation.** The chart's records property now round-trips: assign it back to the same collection variable and a user can edit three phases, click Previous to check something, come back, and their unsaved edits are still on the chart.

*One licensing note: the record-form modal embeds the FlowToolKit Form component, which follows the same paid Flow Tool Kit tiers as the data table - the free tier covers Account, Contact, Case, and Lead. The timeline chart itself is free on every object.*

## Use Cases Beyond Grants

**Program phases.** Child Campaigns under a parent Campaign, bars from StartDate and EndDate, progress fill from a percent field, color by Status. The swimlane layout shows every program's phases at a glance.

**Volunteer scheduling.** Shifts or job schedules as bars grouped by location or role, with the today line marking where the week stands.

**Compact deadline strips.** The Compact density renders thin pill bars in a flat list - a lightweight "what is due when" widget for the top of any operational Flow.

![Compact flat chained list](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/18-timeline-compact-chained.png)

**Deadline boards.** Milestone-only mode (just a date field, no bars) gives one diamond per record - reports due, renewals coming, follow-ups scheduled.

## Get Started

Version 0.6.0.1 is live. Install links and the full changelog are on the [latest release](https://github.com/common-unite/flowtoolkit_chart_js_Public/releases/latest) page. The [Timeline (Gantt) guide](https://github.com/common-unite/flowtoolkit_chart_js_Public/blob/main/documents/TIMELINE_GANTT.md) covers every parameter with configuration considerations and six recipe combinations, from grouped milestones to the deadline-only chained schedule.

The one prerequisite is the [Flow Tool Kit](https://github.com/common-unite/Flow_Tool_Kit_Public) base package. Both packages are free and open source; if you build something with them, file feedback in the Issues tab - this entire release traces back to one community question, and the roadmap runs on what people actually need.
