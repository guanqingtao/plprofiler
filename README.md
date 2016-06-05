PL Profiler extension
=====================

This extension implements performance profiling for PLpgSQL code.

Supported PostgreSQL versions
---------------------------------------------------------------------------------
plprofiler strives to support as many community-supported major versions of
Postgres as reasonable. Currently, the following versions of PostgreSQL are
supported:

9.2 - 9.5

Usage
=================================================================================
Once installed, creates the following objects.

pl_profiler_linestats_current view
----------------------------------

Summarizes profile of each function executed. Each entry represents a line of
PLpgSQL code in a function. This view shows the current content of the data
currently collected in memory for this session only.

+---------------------+------------------+---------------------------------------------------------------------+
| Name                | Type             | Description                                                         |
+=====================+==================+=====================================================================+
| func_oid            | oid              | OID of the function profiled                                        |
+---------------------+------------------+---------------------------------------------------------------------+
| line_number         | bigint           | line of the source within the function                              |
+---------------------+------------------+---------------------------------------------------------------------+
| line                | text             | Text of the line of function code                                   |
+---------------------+------------------+---------------------------------------------------------------------+
| exec_count          | bigint           | Number of times executed                                            |
+---------------------+------------------+---------------------------------------------------------------------+
| total_time          | bigint           | Total time spent in execution, in microseconds                      |
+---------------------+------------------+---------------------------------------------------------------------+
| longest_time        | bigint           | The time in microseconds of the longest time the line was executed  |
+---------------------+------------------+---------------------------------------------------------------------+

pl_profiler_callgraph_current view
----------------------------------

Summarizes function call nesting and the times spent in each of the
call paths. The resulting data is similar to a perf(1) profile. This
view shows the current content of the data currently collected in
memory for this session only.

+---------------------+------------------+---------------------------------------------------------------------+
| Name                | Type             | Description                                                         |
+=====================+==================+=====================================================================+
| stack               | text[]           | The call levels in the form 'funcname(args)'                        |
+---------------------+------------------+---------------------------------------------------------------------+
| call_count          | bigint           | Number of times this call stack was observed.                       |
+---------------------+------------------+---------------------------------------------------------------------+
| us_total            | bigint           | Total time in microsecond spent in this call stack.                 |
+---------------------+------------------+---------------------------------------------------------------------+
| us_children         | bigint           | Time in microseconds spent in subsequent call stacks.               |
+---------------------+------------------+---------------------------------------------------------------------+
| us_total            | bigint           | Time in microseconds spent in this call stack alone.                |
+---------------------+------------------+---------------------------------------------------------------------+

pl_profiler_linestats view
--------------------------

PL Profiler can be configured to be enabled in all sessions and
periodically save the collected stats in a permanent table. This
view aggregates the collected data over from all sessions.

pl_profiler_callgraph view
--------------------------

Equivalent to pl_profiler_linestats for call graph data.

pl_profiler_linestats_data table
--------------------------------

The permanent table where line stats data should be saved for the
pl_profiler_linestats view.

pl_profiler_callgraph_data table
--------------------------------

The permanent table where call graph data should be saved for the
pl_profiler_linestats view.

pl_profiler_save_stats() function
---------------------------------

Function to save the current in memory collected data into the
configured permanent tables.

pl_profiler_reset() function
----------------------------

Can be called by users to reset the contents of the pl_profiler view

pl_profiler_enable(enabled bool) function
-----------------------------------------

Used to enable/disable the profiler for the current session

Configuration
=================================================================================

The following configuration options can be set in the postgresql.conf
file to control the behavior of PL Profiler:

`*plprofiler.enabled*` = true/false # Used to enable/disable the profiler in all sessions. Note that for this GUC to work inside a session, the plprofiler must have been loaded via shared_preload_libraries. If that isn't done, use pl_profiler_enable(true) instead.

`*plprofiler.save_interval*` = seconds # If configured, the profiler will automatically
call the pl_profiler_save_stats() function the first time, it collects data after this
interval has elapsed. The default of 0 seconds turns auto-save off.

`*plprofiler.save_linestats_table*` = 'tablename' # Sets the table name
where pl_profiler_save_stats() inserts the current internal counters
for linestats.
Default is 'pl_profiler_linestats_data'. This table must be in the same
namespace as the plprofiler extension.

`*plprofiler.save_callgraph_table*` = 'tablename' # Sets the table name
where pl_profiler_save_stats() inserts the current internal counters
for callgraph.
Default is 'pl_profiler_callgraph_data'. This table must be in the same
namespace as the plprofiler extension.

License
=================================================================================
plprofiler is released under the Artistic License.

    http://www.opensource.org/licenses/artistic-license.php
