# defects4j-instrumented

This repository includes the instrumented defects4j subjects. We manually transformed the information from the bug report into an oracle, which is added to the buggy version of each subject.

Each modification is tagged as: `D4J_<project>_<bugID>_BUGGY_VERSION_INSTRUMENTED`

The zipped git archives are included in [instrumented-archives](./instrumented-archives).

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
+       if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
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

## Progress

The following list includes the already covered subjects. **Total count: 31** subjects.

* 2 ARJA cannot produce a plausible patch
* 19 ARJA can generate a plausible but incorrect patch
* 10 ARJA can produce correct patch.


<details>
<summary><b>Lang-19</b> (Example; ARJA implausible)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`
* [lang_19.diff](./instrumented-diffs/lang_19.diff)

</details>

<details>
<summary><b>Lang-7</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-822
* new tag: `D4J_Lang_7_BUGGY_VERSION_INSTRUMENTED`
* [lang_7.diff](./instrumented-diffs/lang_7.diff)

</details>

<details>
<summary><b>Lang-16</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-746
* new tag: `D4J_Lang_16_BUGGY_VERSION_INSTRUMENTED`
* [lang_16.diff](./instrumented-diffs/lang_16.diff)

</details>

<details>
<summary><b>Lang-20</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-703
* new tag: `D4J_Lang_20_BUGGY_VERSION_INSTRUMENTED`
* [lang_20.diff](./instrumented-diffs/lang_20.diff)

</details>

<details>
<summary><b>Lang-22</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-662
* new tag: `D4J_Lang_22_BUGGY_VERSION_INSTRUMENTED`
* [lang_22.diff](./instrumented-diffs/lang_22.diff)

</details>

<details>
<summary><b>Lang-35</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-571
* new tag: `D4J_Lang_35_BUGGY_VERSION_INSTRUMENTED`
* [lang_35.diff](./instrumented-diffs/lang_35.diff)

</details>

<details>
<summary><b>Lang-39</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-552
* new tag: `D4J_Lang_39_BUGGY_VERSION_INSTRUMENTED`
* [lang_39.diff](./instrumented-diffs/lang_39.diff)

</details>

<details>
<summary><b>Lang-41</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-535
* new tag: `D4J_Lang_41_BUGGY_VERSION_INSTRUMENTED`
* [lang_41.diff](./instrumented-diffs/lang_41.diff)

</details>

<details>
<summary><b>Lang-45</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-419
* new tag: `D4J_Lang_45_BUGGY_VERSION_INSTRUMENTED`
* [lang_45.diff](./instrumented-diffs/lang_45.diff)

</details>

<details>
<summary><b>Lang-46</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-421
* new tag: `D4J_Lang_46_BUGGY_VERSION_INSTRUMENTED`
* [lang_46.diff](./instrumented-diffs/lang_46.diff)

</details>

<details>
<summary><b>Lang-50</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-368
* new tag: `D4J_Lang_50_BUGGY_VERSION_INSTRUMENTED`
* [lang_50.diff](./instrumented-diffs/lang_50.diff)

</details>



<details>
<summary><b>Math-2</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-1021
* new tag: `D4J_Math_2_BUGGY_VERSION_INSTRUMENTED`
* [math_2.diff](./instrumented-diffs/math_2.diff)

</details>

<details>
<summary><b>Math-8</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-942
* new tag: `D4J_Math_8_BUGGY_VERSION_INSTRUMENTED`
* [math_8.diff](./instrumented-diffs/math_8.diff)

</details>

<details>
<summary><b>Math-20</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-864
* new tag: `D4J_Math_20_BUGGY_VERSION_INSTRUMENTED`
* [math_20.diff](./instrumented-diffs/math_20.diff)

</details>

<details>
<summary><b>Math-22</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-859
* new tag: `D4J_Math_22_BUGGY_VERSION_INSTRUMENTED`
* [math_22.diff](./instrumented-diffs/math_22.diff)

</details>

<details>
<summary><b>Math-31</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-718
* new tag: `D4J_Math_31_BUGGY_VERSION_INSTRUMENTED`
* [math_31.diff](./instrumented-diffs/math_31.diff)

</details>

<details>
<summary><b>Math-39</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-227
* new tag: `D4J_Math_39_BUGGY_VERSION_INSTRUMENTED`
* [math_39.diff](./instrumented-diffs/math_39.diff)

</details>

<details>
<summary><b>Math-49</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-645
* new tag: `D4J_Math_49_BUGGY_VERSION_INSTRUMENTED`
* [math_49.diff](./instrumented-diffs/math_49.diff)

</details>

<details>
<summary><b>Math-50</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-631
* new tag: `D4J_Math_50_BUGGY_VERSION_INSTRUMENTED`
* [math_50.diff](./instrumented-diffs/math_50.diff)

</details>

<details>
<summary><b>Math-53</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-618
* new tag: `D4J_Math_53_BUGGY_VERSION_INSTRUMENTED`
* [math_53.diff](./instrumented-diffs/math_53.diff)

</details>

<details>
<summary><b>Math-56</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-552
* new tag: `D4J_Math_56_BUGGY_VERSION_INSTRUMENTED`
* [math_56.diff](./instrumented-diffs/math_56.diff)

</details>

<details>
<summary><b>Math-58</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-519
* new tag: `D4J_Math_58_BUGGY_VERSION_INSTRUMENTED`
* [math_58.diff](./instrumented-diffs/math_58.diff)

</details>

<details>
<summary><b>Math-60</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-414
* new tag: `D4J_Math_60_BUGGY_VERSION_INSTRUMENTED`
* [math_60.diff](./instrumented-diffs/math_60.diff)

</details>

<details>
<summary><b>Math-65</b> (ARJA implausible)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-377
* new tag: `D4J_Math_65_BUGGY_VERSION_INSTRUMENTED`
* [math_65.diff](./instrumented-diffs/math_65.diff)

</details>

<details>
<summary><b>Math-70</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-369
* new tag: `D4J_Math_70_BUGGY_VERSION_INSTRUMENTED`
* [math_70.diff](./instrumented-diffs/math_70.diff)

</details>

<details>
<summary><b>Math-71</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-358
* new tag: `D4J_Math_71_BUGGY_VERSION_INSTRUMENTED`
* [math_71.diff](./instrumented-diffs/math_71.diff)

</details>

<details>
<summary><b>Math-73</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-343
* new tag: `D4J_Math_73_BUGGY_VERSION_INSTRUMENTED`
* [math_73.diff](./instrumented-diffs/math_73.diff)

</details>

<details>
<summary><b>Math-74</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-338
* new tag: `D4J_Math_74_BUGGY_VERSION_INSTRUMENTED`
* [math_74.diff](./instrumented-diffs/math_74.diff)

</details>

<details>
<summary><b>Math-81</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-308
* new tag: `D4J_Math_81_BUGGY_VERSION_INSTRUMENTED`
* [math_81.diff](./instrumented-diffs/math_81.diff)

</details>

<details>
<summary><b>Math-95</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-227
* new tag: `D4J_Math_95_BUGGY_VERSION_INSTRUMENTED`
* [math_95.diff](./instrumented-diffs/math_95.diff)

</details>

<details>
<summary><b>Math-98</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-209
* new tag: `D4J_Math_98_BUGGY_VERSION_INSTRUMENTED`
* [math_98.diff](./instrumented-diffs/math_98.diff)

</details>


<details>
<summary><b>Math-103</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-167
* new tag: `D4J_Math_103_BUGGY_VERSION_INSTRUMENTED`
* [math_103.diff](./instrumented-diffs/math_103.diff)

</details>




## Not Supported Subjects

The following list includes subjects, for which the bug report does not contain sufficient information to formulate a meaningful assertion. **Total count: 10** subject.

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
<summary><b>Math-68</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-362

> LevenbergMarquardtOptimizer ignores the VectorialConvergenceChecker parameter passed to it. This makes it hard to specify custom stopping criteria for the optimizer.

→ The bug report does not provide enough information for an assertion.

</details>

<details>
<summary><b>Math-80</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-318

> Some results computed by EigenDecompositionImpl are wrong. The following case computed by Fortran Lapack fails with version 2.0

→ only one failing test case, which is already added to the test suite in Defects4J

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
<summary><b>TODO Lang-43</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-477

→ needs check for OutOfMemoryError

→ code cannot be instrumented at the source code level because the method is defined in an non-accessible super class. Override is not possible because final.

</details>
