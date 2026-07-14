# Interactive Charts in Flow Screens - No Code Required

Most Salesforce admins have built a Flow that collects data and then... hands it to someone else to turn into a spreadsheet. What if the chart just lived on the screen?

Flow Tool Kit: Chart.js is a free, open-source Salesforce extension package that brings Chart.js into Flow Screens as a fully configurable component. You pick the records, choose how to group and aggregate them, and the chart renders. Click a wedge and it drives every other component on the screen. No Apex, no LWC, no custom code - everything happens in the property editor.

![Multi-chart dashboard built in a Flow Screen](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-dashboard-multi.png)

---

## Overview

If you coordinate volunteers, manage grants, track fundraising campaigns, or run any program where you need to make sense of records in Salesforce, you have probably wanted a chart without wanting to build one. Dashboards exist, but they live separately from your operational Flows. Reports are great, but they require a separate tab, separate context, separate mental gear-shift.

This component puts a live chart directly on a Flow Screen - the same screen where your team enters data, routes approvals, or runs a guided process. It reads from any record collection variable you already have in the Flow. The chart updates when the data changes. Users can click into it and see the underlying records without navigating anywhere.

It is built as an extension to the [Flow Tool Kit](https://github.com/common-unite/Flow_Tool_Kit_Public) base package, which provides the shared property-editor framework. If you are already a FlowToolKit user, installation is one additional package link.

---

## How It Works

The component is called **Form (Chart)**. You find it in the Flow Builder component panel and drag it onto a Screen element like any other screen component.

![Form (Chart) in Flow Builder](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-flow-builder-screen.png)

Once it is on the screen, the property editor on the right side of Flow Builder is where all configuration happens. There are three decisions to make:

**Source Records** - pick any record collection variable from your Flow. The chart auto-detects the SObject type and filters every field dropdown below to fields on that object. You do not type field API names; you pick from a filtered list.

**Chart type and aggregation** - choose Bar, Line, Area, Pie, or Doughnut. Then decide whether you want to aggregate (Sum, Count, Average, Min, Max - grouped by a field you choose) or plot one point per record (detail mode, useful for ranking or ordering). For date fields, a Date Bucket picker appears: Day, Week, Month, Quarter, or Year. The labels render as human-friendly formats - `Mar 2026`, `Q1 2026`, `Week 18, 2026` - using the running user's locale.

**Appearance** - an accordion of optional configuration: value labels on bars and slices, legend position, smooth curves on line and area charts, grid lines, and colors. The default color comes from the org's SLDS brand token, so it matches your Experience Cloud site out of the box. If you want per-value colors - Closed Won is green, Closed Lost is red - there is a point-and-click mapping UI for that.

![Full property editor](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-cpe-full-area.png)

Save the Flow and run it. The chart is live.

---

## Use Cases

**Fundraising dashboards.** A doughnut chart showing Sum of Amount by Stage, side-by-side with an area chart of the same data bucketed by Close Date month. Both reading from the same Opportunities collection. Click a stage in the doughnut and the area chart re-renders for just those records.

**Volunteer attendance.** Group volunteer hours by month or by shift type. An area chart shows the trend; a horizontal bar chart shows which programs draw the most hours. If you run Volunteers for Salesforce, the same record collections you already query in your volunteer Flows can feed straight into a chart.

![Area chart by month](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-area-by-month.png)

Here is the same idea running on a volunteer portal profile page. The chart reads the logged-in user's `GW_Volunteers__Hours_Worked__c` records, buckets them by Planned Start Date (Month), and renders them as a horizontal bar chart right above the profile-update fields. Tooltips show the exact sum for each month on hover.

![Volunteer portal profile page with a "My Past Volunteer Hours" chart by month](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/chart-volunteer-portal-profile-2.gif)

![Walkthrough of the same volunteer portal profile chart](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js_Public/main/documents/screenshots/chart-volunteer-portal-profile-1.gif)

**Grant pipeline.** A bar chart of open grant applications by stage, or a pie chart of awarded versus declined. Program managers can see where things stand without leaving the Flow they use to update records.

**Capacity planning.** A horizontal bar chart of available volunteer shifts by location or by date, where long location names fit cleanly on the label axis.

![Horizontal bar with tooltip](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-bar-horizontal-tooltip.png)

**Boolean segmentation.** Group any dataset by a Boolean field - attended / did not attend, completed / incomplete, closed / open - and the chart renders Yes and No labels correctly. Salesforce omits false-valued Booleans from record JSON in some contexts; the component handles that case so false values land in the No bucket rather than disappearing.

![Pie chart grouped by a Boolean field](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-pie-boolean.png)

**Top N rankings.** Sort your upstream Get Records by a numeric field descending, set Limit to 10 in the property editor, and the chart shows the top 10 records. Useful for donor rankings, largest accounts, highest-volume programs.

---

## Flow Features and Considerations

A few things worth knowing before you build.

**This is a Screen component, not a standalone app.** It lives inside a Flow Screen element alongside your other components. That is a feature, not a limitation - it means the chart can read from the same record collections your other components use, and its click outputs can drive those same components. But it does mean you need a Flow Screen to host it.

**The FlowToolKit base package is required.** This component is an extension package that depends on [Flow Tool Kit](https://github.com/common-unite/Flow_Tool_Kit_Public). Install the base package first, then install this one. If you are already using FlowToolKit's Form Builder or other components, you are set.

**Upstream Get Records drives everything.** The chart does not query Salesforce directly. You bring the records in through a Get Records element (or a subflow output, or an Apex action - any record collection variable works). This means you control the filter, the sort order, and the fields returned. The chart renders what Flow hands it.

**Performance scales with collection size.** Up to ~500 records renders sub-second. 500 to 2,000 is noticeable but acceptable. Above 2,000, consider whether you need that many records on a Screen element. Use Get Records filters to keep the input lean. Also verify that Lightning Web Security is enabled in your org (Setup → Session Settings → "Use Lightning Web Security for Lightning web components") - the legacy Locker Service implementation wraps every object in a heavyweight proxy that degrades significantly at scale; LWS avoids that.

**Screen Actions and reactive screens.** The component participates in Flow's reactive screen architecture. Inputs bound to Flow variables update the chart live - no screen reload, no navigation. This is what makes filter-and-chart patterns work: a picklist choice above the chart filters the source collection into a new variable; the chart re-renders the moment the choice changes.

---

## Reactivity - the Part Worth Spending Time On

The chart maintains three output streams at all times: **Selected** (the records behind the clicked element), **Visible** (everything currently rendered), and **Active** (selected when something is clicked, otherwise visible). Active is the right default for most downstream components.

![Doughnut with a wedge selected](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-doughnut-selected.png)

Bind a datatable's record input to `{!Form_Chart_1.activeRecords}`. On initial load the table shows everything. The moment a user clicks a wedge, the table filters to that group's records. Click the wedge again to deselect and the table returns to the full set. No navigation, no screen reload - the whole interaction happens in place.

The same wiring works between two charts. Drop a summary chart (Sum of Amount by Stage) and a detail chart (Sum of Amount by Owner) on the same Screen. Bind the detail chart's Source Records to the summary chart's `activeRecords`. Click a stage in the summary and the detail chart re-renders for that stage's records only. That is a functional drill-down dashboard built entirely in Flow Builder.

![Multi-chart dashboard](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-dashboard-multi.png)

The reactivity extends to inputs too. Bind the chart's Aggregate Function to a choice component and users can toggle between Count and Sum live. Bind Group By Field to a choice and the chart pivots between groupings without leaving the screen. These are the kinds of interactions that used to require a custom LWC; here they are wired together in the property editor.

Per-grouping colors are part of the same story. When your org has meaningful color conventions - a stage or status field where specific values should always appear as specific colors - the color mapping UI lets you assign those without writing JSON or touching code.

![Per-grouping color picker](https://raw.githubusercontent.com/common-unite/flowtoolkit_chart_js/master/documents/screenshots/chart-cpe-colors-per-grouping.png)

---

## Get Started

The package is free and open source. The source lives at [github.com/common-unite/flowtoolkit_chart_js](https://github.com/common-unite/flowtoolkit_chart_js), along with a Quick Start guide, a full Property Editor Reference, output documentation, and a recipes page covering the patterns described above.

The one prerequisite is the [Flow Tool Kit](https://github.com/common-unite/Flow_Tool_Kit_Public) base package. If you are already using FlowToolKit, installation is straightforward. If you are new to the ecosystem, the base package is also free.

If you build something with it, file feedback in the Issues tab. The roadmap is public and features get prioritized based on what people actually need.
