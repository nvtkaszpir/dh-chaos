-   [Summary](#summary)
-   [Experiment](#experiment)
    -   [wordpress-is-alive](#wordpress-is-alive)
        -   [Summary](#summary-1)
        -   [Definition](#definition)
        -   [Result](#result)
        -   [Appendix](#appendix)

\newpage
Summary
=======

This report aggregates 1 experiments spanning over the following subjects:

\newpage

Experiment
==========

wordpress-is-alive
------------------

N/A

### Summary

wordpress-is-alive

N/A

|                   |                                                             |
|-------------------|-------------------------------------------------------------|
| **Status**        | completed                                                   |
| **Tagged**        |                                                             |
| **Executed From** | misiek                                                      |
| **Platform**      | Linux-4.16.3-041603-generic-x86\_64-with-debian-stretch-sid |
| **Started**       | Sat, 07 Sep 2019 15:56:42 GMT                               |
| **Completed**     | Sat, 07 Sep 2019 16:00:04 GMT                               |
| **Duration**      | 3 minutes                                                   |

### Definition

The experiment was made of 2 actions, to vary conditions in your system, and 0 probes, to collect objective data from your system during the experiment.

#### Steady State Hypothesis

The steady state hypothesis this experiment tried was “**wordpress-endpoint-exists**”.

##### Before Run

The steady state was verified

| Probe                                 | Tolerance | Verified |
|---------------------------------------|-----------|----------|
| microservice\_available\_and\_healthy |  True     | True     |

##### After Run

The steady state was verified

| Probe                                 | Tolerance | Verified |
|---------------------------------------|-----------|----------|
| microservice\_available\_and\_healthy |  True     | True     |

#### Method

The experiment method defines the sequence of activities that help gathering evidence towards, or against, the hypothesis.

The following activities were conducted as part of the experimental's method:

| Type   | Name             |
|--------|------------------|
| action |  terminate\_pods |
| action |  drain\_nodes    |

### Result

The experiment was conducted on Sat, 07 Sep 2019 15:56:42 GMT and lasted roughly 3 minutes.

#### Action - terminate\_pods

|                  |                               |
|------------------|-------------------------------|
| **Status**       | succeeded                     |
| **Background**   | False                         |
| **Started**      | Sat, 07 Sep 2019 15:56:42 GMT |
| **Ended**        | Sat, 07 Sep 2019 15:56:42 GMT |
| **Duration**     | 0 seconds                     |
| **Paused After** | 90s                           |

The action provider that was executed:

<table>
<colgroup>
<col width="21%" />
<col width="78%" />
</colgroup>
<tbody>
<tr class="odd">
<td align="left"><strong>Type</strong></td>
<td align="left">python</td>
</tr>
<tr class="even">
<td align="left"><strong>Module</strong></td>
<td align="left">chaosk8s.pod.actions</td>
</tr>
<tr class="odd">
<td align="left"><strong>Function</strong></td>
<td align="left">terminate_pods</td>
</tr>
<tr class="even">
<td align="left"><strong>Arguments</strong></td>
<td align="left">{'label_selector': 'app=wp-02-wordpress', 'name_pattern': None, 'all': False, 'rand': True, 'mode': 'fixed', 'qty': 2, 'grace_period': -1, 'ns': 'default', 'order': 'alphabetic'}</td>
</tr>
</tbody>
</table>

#### Action - drain\_nodes

|                  |                               |
|------------------|-------------------------------|
| **Status**       | succeeded                     |
| **Background**   | False                         |
| **Started**      | Sat, 07 Sep 2019 15:58:12 GMT |
| **Ended**        | Sat, 07 Sep 2019 15:58:33 GMT |
| **Duration**     | 21 seconds                    |
| **Paused After** | 90s                           |

The action provider that was executed:

|               |                                     |
|---------------|-------------------------------------|
| **Type**      | python                              |
| **Module**    | chaosk8s.node.actions               |
| **Function**  | drain\_nodes                        |
| **Arguments** | {'label\_selector': 'app=drain-me'} |

### Appendix

#### Action - terminate\_pods

The *action* returned the following result:

``` javascript
None
```

#### Action - drain\_nodes

The *action* returned the following result:

``` javascript
True
```
