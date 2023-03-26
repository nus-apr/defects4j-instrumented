# defects4j-instrumented

This repository includes the instrumented defects4j subjects. We manually transformed the information from the bug report into an oracle, which is added to the buggy version of each subject.

Each modification is tagged as: `D4J_<project>_<bugID>_BUGGY_VERSION_INSTRUMENTED`

The zipped git archives are included in [instrumented-archives](./instrumented-archives).

The diff files are included in [instrumented-diffs](./instrumented-diffs).

*Note: For now, we will only have direct modifications of the buggy source code. Later, we want to introduce an automated mechanism that (given the source code modification) directly instruments the Java bytecode.*

## Current general issues

* not all test cases call the method mentioned in the bug report; they call the underlying buggy method
* with the additional specificaton from the bug report, other test cases can also fail. They should be fixed with the correct bug though. E.g., Lang-7, a test case interpretes both (the incorrect and the correct) behavior a suitable way so that the test cases passes; but with our instrumentation it fails because we throw a RuntimeException. Correct: throw NumberFormatException. Buggy: return null. The other test case handles both but not our RuntimeException.

## Methodology: Subject Selection

* we start with subjects, for which ARJA can generated plausible patch
* ARJA
	* considers 224 bugs from Defects4J
	* plausible patches: 59
	* correct patches: 18

## Methodology: Instrumentation Writing

1. TODO

## Example: Lang-19

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
index d84aae58..f5757eac 100644
--- a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
@@ -465,7 +465,23 @@ public class StringEscapeUtils {
      * @since 3.0
      */
     public static final String unescapeHtml4(String input) {
-        return UNESCAPE_HTML4.translate(input);
+       if (Boolean.parseBoolean(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                // Original Code START
+                return UNESCAPE_HTML4.translate(input);
+                // Original Code END
+            } catch (StringIndexOutOfBoundsException e) {
+                if (input.contains("&")) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+       } else {
+            // Original Code START
+            return UNESCAPE_HTML4.translate(input);
+            // Original Code END
+       }
     }
```

Additionally, we insert the comment `// defects4j.instrumentation` at the end of each line of the oracle's instrumentation. This is done so that an automated tool can detect the oracle and, e.g., avoid a repair engine removing the oracle during the patch generation.

## Instrumented Subjects Overview (43 + 3)

ARJA reported that it can generate plausible patches for 59 subjects. From these 59, we instrumented **in total 43** subjects. From these 43,

* 29 subjects are covered by ARJA with plausible but incorrect patches, and
* 14 subjects are covered by ARJA with correct patches.

For **16** subjects we were not able to write an oracle (see [below](#not-supported-arja-plausible-subjects-17)).

Additionally, we instrumented **2** subjects, for which ARJA cannot generate any plausible patch (see [below](#other-supported-subjects-2)).



### commons-lang (16 subjects)

<details>
<summary><b>Lang-7</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-822
* new tag: `D4J_Lang_7_BUGGY_VERSION_INSTRUMENTED`
* [lang_7.zip](./instrumented-archives/lang_7.zip), [lang_7.diff](./instrumented-diffs/lang_7.diff)

</details>

<details>
<summary><b>Lang-16</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-746
* new tag: `D4J_Lang_16_BUGGY_VERSION_INSTRUMENTED`
* [lang_16.zip](./instrumented-archives/lang_16.zip), [lang_16.diff](./instrumented-diffs/lang_16.diff)

</details>

<details>
<summary><b>Lang-20</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-703
* new tag: `D4J_Lang_20_BUGGY_VERSION_INSTRUMENTED`
* [lang_20.zip](./instrumented-archives/lang_20.zip), [lang_20.diff](./instrumented-diffs/lang_20.diff)

</details>

<details>
<summary><b>Lang-22</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-662
* new tag: `D4J_Lang_22_BUGGY_VERSION_INSTRUMENTED`
* [lang_22.zip](./instrumented-archives/lang_22.zip), [lang_22.diff](./instrumented-diffs/lang_22.diff)

</details>

<details>
<summary><b>Lang-35</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-571
* new tag: `D4J_Lang_35_BUGGY_VERSION_INSTRUMENTED`
* [lang_35.zip](./instrumented-archives/lang_35.zip), [lang_35.diff](./instrumented-diffs/lang_35.diff)

</details>

<details>
<summary><b>Lang-39</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-552
* new tag: `D4J_Lang_39_BUGGY_VERSION_INSTRUMENTED`
* [lang_39.zip](./instrumented-archives/lang_39.zip), [lang_39.diff](./instrumented-diffs/lang_39.diff)

</details>

<details>
<summary><b>Lang-41</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-535
* new tag: `D4J_Lang_41_BUGGY_VERSION_INSTRUMENTED`
* [lang_41.zip](./instrumented-archives/lang_41.zip), [lang_41.diff](./instrumented-diffs/lang_41.diff)

</details>

<details>
<summary><b>Lang-43</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-477
* new tag: `D4J_Lang_43_BUGGY_VERSION_INSTRUMENTED`
* [lang_43.zip](./instrumented-archives/lang_43.zip), [lang_43.diff](./instrumented-diffs/lang_43.diff)

</details>

<details>
<summary><b>Lang-45</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-419
* new tag: `D4J_Lang_45_BUGGY_VERSION_INSTRUMENTED`
* [lang_45.zip](./instrumented-archives/lang_45.zip), [lang_45.diff](./instrumented-diffs/lang_45.diff)

</details>

<details>
<summary><b>Lang-46</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-421
* new tag: `D4J_Lang_46_BUGGY_VERSION_INSTRUMENTED`
* [lang_46.zip](./instrumented-archives/lang_46.zip), [lang_46.diff](./instrumented-diffs/lang_46.diff)

</details>

<details>
<summary><b>Lang-50</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-368
* new tag: `D4J_Lang_50_BUGGY_VERSION_INSTRUMENTED`
* [lang_50.zip](./instrumented-archives/lang_50.zip), [lang_50.diff](./instrumented-diffs/lang_50.diff)

</details>

<details>
<summary><b>Lang-51</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-365
* new tag: `D4J_Lang_51_BUGGY_VERSION_INSTRUMENTED`
* [lang_51.zip](./instrumented-archives/lang_51.zip), [lang_51.diff](./instrumented-diffs/lang_51.diff)

</details>

<details>
<summary><b>Lang-55</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-315
* new tag: `D4J_Lang_55_BUGGY_VERSION_INSTRUMENTED`
* [lang_55.zip](./instrumented-archives/lang_55.zip), [lang_55.diff](./instrumented-diffs/lang_55.diff)

</details>

<details>
<summary><b>Lang-59</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-299
* new tag: `D4J_Lang_59_BUGGY_VERSION_INSTRUMENTED`
* [lang_59.zip](./instrumented-archives/lang_59.zip), [lang_59.diff](./instrumented-diffs/lang_59.diff)

</details>

<details>
<summary><b>Lang-61</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-294
* new tag: `D4J_Lang_61_BUGGY_VERSION_INSTRUMENTED`
* [lang_61.zip](./instrumented-archives/lang_61.zip), [lang_61.diff](./instrumented-diffs/lang_61.diff)

</details>

<details>
<summary><b>Lang-63</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-281
* new tag: `D4J_Lang_63_BUGGY_VERSION_INSTRUMENTED`
* [lang_63.zip](./instrumented-archives/lang_63.zip), [lang_63.diff](./instrumented-diffs/lang_63.diff)

</details>


### commons-math (20 subjects)

<details>
<summary><b>Math-2</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-1021
* new tag: `D4J_Math_2_BUGGY_VERSION_INSTRUMENTED`
* [math_2.zip](./instrumented-archives/math_2.zip), [math_2.diff](./instrumented-diffs/math_2.diff)

</details>

<details>
<summary><b>Math-8</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-942
* new tag: `D4J_Math_8_BUGGY_VERSION_INSTRUMENTED`
* [math_8.zip](./instrumented-archives/math_8.zip), [math_8.diff](./instrumented-diffs/math_8.diff)

</details>

<details>
<summary><b>Math-20</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-864
* new tag: `D4J_Math_20_BUGGY_VERSION_INSTRUMENTED`
* [math_20.zip](./instrumented-archives/math_20.zip), [math_20.diff](./instrumented-diffs/math_20.diff)

</details>

<details>
<summary><b>Math-22</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-859
* new tag: `D4J_Math_22_BUGGY_VERSION_INSTRUMENTED`
* [math_22.zip](./instrumented-archives/math_22.zip), [math_22.diff](./instrumented-diffs/math_22.diff)

</details>

<details>
<summary><b>Math-31</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-718
* new tag: `D4J_Math_31_BUGGY_VERSION_INSTRUMENTED`
* [math_31.zip](./instrumented-archives/math_31.zip), [math_31.diff](./instrumented-diffs/math_31.diff)

</details>

<details>
<summary><b>Math-39</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-227
* new tag: `D4J_Math_39_BUGGY_VERSION_INSTRUMENTED`
* [math_39.zip](./instrumented-archives/math_39.zip), [math_39.diff](./instrumented-diffs/math_39.diff)

</details>

<details>
<summary><b>Math-49</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-645
* new tag: `D4J_Math_49_BUGGY_VERSION_INSTRUMENTED`
* [math_49.zip](./instrumented-archives/math_49.zip), [math_49.diff](./instrumented-diffs/math_49.diff)

</details>

<details>
<summary><b>Math-53</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-618
* new tag: `D4J_Math_53_BUGGY_VERSION_INSTRUMENTED`
* [math_53.zip](./instrumented-archives/math_53.zip), [math_53.diff](./instrumented-diffs/math_53.diff)

</details>

<details>
<summary><b>Math-58</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-519
* new tag: `D4J_Math_58_BUGGY_VERSION_INSTRUMENTED`
* [math_58.zip](./instrumented-archives/math_58.zip), [math_58.diff](./instrumented-diffs/math_58.diff)

</details>

<details>
<summary><b>Math-60</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-414
* new tag: `D4J_Math_60_BUGGY_VERSION_INSTRUMENTED`
* [math_60.zip](./instrumented-archives/math_60.zip), [math_60.diff](./instrumented-diffs/math_60.diff)

</details>

<details>
<summary><b>Math-70</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-369
* new tag: `D4J_Math_70_BUGGY_VERSION_INSTRUMENTED`
* [math_70.zip](./instrumented-archives/math_70.zip), [math_70.diff](./instrumented-diffs/math_70.diff)

</details>

<details>
<summary><b>Math-74</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-338
* new tag: `D4J_Math_74_BUGGY_VERSION_INSTRUMENTED`
* [math_74.zip](./instrumented-archives/math_74.zip), [math_74.diff](./instrumented-diffs/math_74.diff)

</details>

<details>
<summary><b>Math-95</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-227
* new tag: `D4J_Math_95_BUGGY_VERSION_INSTRUMENTED`
* [math_95.zip](./instrumented-archives/math_95.zip), [math_95.diff](./instrumented-diffs/math_95.diff)

</details>

<details>
<summary><b>Math-98</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-209
* new tag: `D4J_Math_98_BUGGY_VERSION_INSTRUMENTED`
* [math_98.zip](./instrumented-archives/math_98.zip), [math_98.diff](./instrumented-diffs/math_98.diff)

</details>

<details>
<summary><b>Math-103</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-167
* new tag: `D4J_Math_103_BUGGY_VERSION_INSTRUMENTED`
* [math_103.zip](./instrumented-archives/math_103.zip), [math_103.diff](./instrumented-diffs/math_103.diff)

</details>


### joda-time (4 subjects)

<details>
<summary><b>Time-4</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://github.com/JodaOrg/joda-time/issues/88
* new tag: `D4J_Time_4_BUGGY_VERSION_INSTRUMENTED`
* [time_4.zip](./instrumented-archives/time_4.zip), [time_4.diff](./instrumented-diffs/time_4.diff)

</details>

<details>
<summary><b>Time-11</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://github.com/JodaOrg/joda-time/issues/18
* new tag: `D4J_Time_11_BUGGY_VERSION_INSTRUMENTED`
* [time_11.zip](./instrumented-archives/time_11.zip), [time_11.diff](./instrumented-diffs/time_11.diff)

</details>

<details>
<summary><b>Time-14</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://sourceforge.net/p/joda-time/bugs/151
* new tag: `D4J_Time_14_BUGGY_VERSION_INSTRUMENTED`
* [time_14.zip](./instrumented-archives/time_14.zip), [time_14.diff](./instrumented-diffs/time_14.diff)

</details>


### jfreechart (3 subjects)

<details>
<summary><b>Chart-1</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://sourceforge.net/p/jfreechart/bugs/983
* new tag: `D4J_Chart_1_BUGGY_VERSION_INSTRUMENTED`
* [chart_1.zip](./instrumented-archives/chart_1.zip), [chart_1.diff](./instrumented-diffs/chart_1.diff)

</details>

<details>
<summary><b>Chart-5</b> (ARJA correct)</summary>

* Bug Report: https://sourceforge.net/p/jfreechart/bugs/862
* new tag: `D4J_Chart_5_BUGGY_VERSION_INSTRUMENTED`
* [chart_5.zip](./instrumented-archives/chart_5.zip), [chart_5.diff](./instrumented-diffs/chart_5.diff)

</details>

<details>
<summary><b>Chart-12</b> (ARJA correct)</summary>

* Bug Report: https://sourceforge.net/p/jfreechart/patches/213
* new tag: `D4J_Chart_12_BUGGY_VERSION_INSTRUMENTED`
* [chart_12.zip](./instrumented-archives/chart_12.zip), [chart_12.diff](./instrumented-diffs/chart_12.diff)

</details>


## Not Supported ARJA Plausible Subjects (16)

The following list includes subjects, for which the bug report does not contain sufficient information to formulate a meaningful assertion. **Total count: 16** subjects.

<details>
<summary><b>Lang-53</b> (ARJA implausible)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-346

→ only one failing test case, which is already added to the test suite in Defects4J

</details>

<details>
<summary><b>Lang-60</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-295

> While fixing ~~[LANG-294](https://issues.apache.org/jira/browse/LANG-294)~~ I noticed that there are two other places in StrBuilder that reference thisBuf.length and unless I'm mistaken they shouldn't.
> 

→ unclear how to formulate an assertion because the reporter just reports that some code needs change without any problem description

</details>

<details>
<summary><b>Math-5</b> (ARJA corret)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-934

→ only one failing test case, which is already added to the test suite in Defects4J

</details>

<details>
<summary><b>Math-6</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-949

> "The method LevenbergMarquardtOptimizer.getIterations() does not report the correct number of iterations; It always returns 0. A quick look at the code shows that only SimplexOptimizer calls BaseOptimizer.incrementEvaluationsCount()
> 
> I've put a test case below. Notice how the evaluations count is correctly incremented, but the iterations count is not."

The bug report says that the method always returns zero but does not say when it is correct and when it is incorrect. It provides a test case; however, this one is already included in the defects4j test suite.

</details>

<details>
<summary><b>Math-28</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-828

> SimplexSolver throws UnboundedSolutionException when trying to solve minimization linear programming problem. The number of exception thrown depends on the number of variables.

The bug report says that the `SimplexSolver` throws an `UnboundedSolutionException,` but it remains unclear when it is expected and when unexpected. So we might not be able to write a general oracle. Also the test throws a different exception: `MaxCountExceededException`. Both extend the `MathIllegalStateException`, which is expected *“if no solution fulfilling the constraints can be found in the allowed number of iterations”*.

</details>

<details>
<summary><b>Math-50</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-631
* new tag: `D4J_Math_50_BUGGY_VERSION_INSTRUMENTED`
* [math_50.zip](./instrumented-archives/math_50.zip), [math_50.diff](./instrumented-diffs/math_50.diff)

→ only one failing test case; a similar one has already been added to the test suite in Defects4J
</details>

<details>
<summary><b>Math-56</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-552
* new tag: `D4J_Math_56_BUGGY_VERSION_INSTRUMENTED`
* [math_56.zip](./instrumented-archives/math_56.zip), [math_56.diff](./instrumented-diffs/math_56.diff)

→ only one failing test case, which has already been added to the test suite in Defects4J
</details>

<details>
<summary><b>Math-68</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-362

> LevenbergMarquardtOptimizer ignores the VectorialConvergenceChecker parameter passed to it. This makes it hard to specify custom stopping criteria for the optimizer.

→ The bug report does not provide enough information for an assertion.

</details>

<details>
<summary><b>Math-71</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-358

→ only one failing test case, which has already been added to the test suite in Defects4J

</details>

<details>
<summary><b>Math-73</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-343

→ The bug report does not provide enough information for a assertion **that is consistent with the developer patch**.

</details>

<details>
<summary><b>Math-80</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-318

> Some results computed by EigenDecompositionImpl are wrong. The following case computed by Fortran Lapack fails with version 2.0

→ only one failing test case, which is already added to the test suite in Defects4J

</details>

<details>
<summary><b>Math-81</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-308
* new tag: `D4J_Math_81_BUGGY_VERSION_INSTRUMENTED`
* [math_81.zip](./instrumented-archives/math_81.zip), [math_81.diff](./instrumented-diffs/math_81.diff)

→ only one failing test case; a similar one has already been added to the test suite in Defects4J
</details>

<details>
<summary><b>Math-82</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-288

→ only one failing test case, which is already added to the test suite in Defects4J

</details>

<details>
<summary><b>Math-84</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-283

> MultiDirectional.iterateSimplex loops forever if the starting point is the correct solution.
> 

However, we cannot check for this. The class allows to set a maximum number of iterations, as done in the provided test case:

```java
multiDirectional.setMaxIterations(100);
multiDirectional.setMaxEvaluations(1000);
```

The provided test throws: `org.apache.commons.math.optimization.OptimizationException: org.apache.commons.math.MaxIterationsExceededException: Maximal number of iterations (100) exceeded`. However, but generally, this does not have to be a bug…

</details>

<details>
<summary><b>Math-85</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-280

The provided test case throws an `ConvergenceException`, which is generally not a bug. Without more information, we cannot formulate a general oracle. Looking at the developer-provided patch it gets clear that a `ConvergenceException` is not acceptable if `fa` or `fb` are `0`, but this information is **not** included in the report.

</details>

<details>
<summary><b>Math-86</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-274

> And it should throw an exception but it does not. I tested the matrix in R and R's cholesky decomposition method returns that the matrix is not symmetric positive definite.
> 

Condition for a check is not known; in fact this check is wrong in the current implementation and needs repair.

</details>

<details>
<summary><b>Chart-3</b> (ARJA correct)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Chart-7</b> (ARJA plausible but incorrect)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Chart-13</b> (ARJA plausible but incorrect)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Chart-15</b> (ARJA plausible but incorrect)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Chart-19</b> (ARJA plausible but incorrect)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Chart-25</b> (ARJA plausible but incorrect)</summary>

* Bug Report: UNKNOWN

</details>

<details>
<summary><b>Time-15</b> (ARJA correct)</summary>

* Bug Report: https://sourceforge.net/p/joda-time/bugs/147

→ only one failing test case, which is already added to the test suite in Defects4J

</details>


## Other Supported Subjects (2)

<details>
<summary><b>Lang-19</b> (Example; ARJA implausible)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`
* [lang_19.zip](./instrumented-archives/lang_19.zip), [lang_19.diff](./instrumented-diffs/lang_19.diff)

</details>

## Additional subjects

<details>
<summary><b>Lang-6</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-857
* new tag: `D4J_Lang_6_BUGGY_VERSION_INSTRUMENTED`
* [lang_6.zip](./instrumented-archives/lang_6.zip), [lang_6.diff](./instrumented-diffs/lang_6.diff)

</details>

<details>
<summary><b>Lang-57</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-304
* new tag: `D4J_Lang_57_BUGGY_VERSION_INSTRUMENTED`
* [lang_57.zip](./instrumented-archives/lang_57.zip), [lang_57.diff](./instrumented-diffs/lang_57.diff)

</details>

<details>
<summary><b>Math-59</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-482
* new tag: `D4J_Math_59_BUGGY_VERSION_INSTRUMENTED`
* [math_59.zip](./instrumented-archives/math_59.zip), [math_59.diff](./instrumented-diffs/math_59.diff)

</details>


## Additional non-supported subjects

<details>
<summary><b>Math-33</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-781

> Methode SimplexSolver.optimeze(...) gives bad results with commons-math3-3.0
> in a simple test problem. It works well in commons-math-2.2.

→ Bug report does not mention specifics. It only compares with a previous version that also does allow the setup of a simple check based on metamorphic relations because the previous version is not available within this codebase.
</details>

<details>
<summary><b>Math-80</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-318

→ the user reports wrong results for the EigenDecompositionImpl but there is no general condition to check. The report mentions the fortran library LAPACK version 3.2.1 which could generally serve as reference implementation, but this would go beyond a simple instrumentation.
</details>

<details>
<summary><b>Math-82</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-288

> SimplexSolver didn't find the optimal solution.

→ Bug report explains a case where the solver does not find the optimal solution. However, there are no constraints or conditions that could be checked in our general oracle beyond the concrete test case.
</details>

<details>
<summary><b>Math-85</b> (ARJA implausible, but in search space of ARJA-e)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-280

> … gives the exception below. It should return (approx) 2.0000…
> 

→ the bug report illustrates multiple examples, for which the NormalDistributionImpl.inverseCumulativeProbability(..) results in a `MathException`. However, the `MathException` is also sometimes expected, as the methods signature actually has the corresponding throw declaration. To define an oracle, we would need to know when it is expected or when it is not expected. But the report does not include this information.
</details>

