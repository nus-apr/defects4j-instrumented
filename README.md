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

The following list includes the already covered subjects. **Total count: 8** subjects.

* 1 ARJA cannot produce a plausible patch
* 5 ARJA can generate a plausible but incorrect patch
* 2 ARJA can produce correct patch.


<details>
<summary><b>Lang-19</b> (Example; ARJA no plausible patch)</summary>

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

</details>

<details>
<summary><b>Lang-7</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-822
* new tag: `D4J_Lang_7_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
index d49da7f4..c3c8bbde 100644
--- a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
+++ b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
@@ -443,6 +443,24 @@ public class NumberUtils {
      * @throws NumberFormatException if the value cannot be converted
      */
     public static Number createNumber(String str) throws NumberFormatException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            Number returnValue = null;
+            try {
+                returnValue = createNumber_original(str);
+            } catch (NumberFormatException e) {
+                throw e;
+            }
+
+            if (str != null && str.startsWith("--") && returnValue == null) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+            return returnValue;
+        } else {
+            return createNumber_original(str);
+        }
+    }
+
+    public static Number createNumber_original(String str) throws NumberFormatException {
         if (str == null) {
             return null;
         }
```

</details>

<details>
<summary><b>Lang-16</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-746
* new tag: `D4J_Lang_16_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
index 882358f2..39bffaeb 100644
--- a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
+++ b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
@@ -442,6 +442,27 @@ public class NumberUtils {
      * @throws NumberFormatException if the value cannot be converted
      */
     public static Number createNumber(String str) throws NumberFormatException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                return createNumber_original(str);
+            } catch (NumberFormatException e) {
+                if (str != null && (str.startsWith("0X") || str.startsWith("-0X"))) {
+                    try {
+                        Integer.decode(str);
+                    } catch (NumberFormatException e_decode) {
+                        throw e;
+                    }
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+        } else {
+            return createNumber_original(str);
+        }
+    }
+
+    public static Number createNumber_original(String str) throws NumberFormatException {
         if (str == null) {
             return null;
         }
```

</details>

<details>
<summary><b>Lang-20</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-703
* new tag: `D4J_Lang_20_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/StringUtils.java b/src/main/java/org/apache/commons/lang3/StringUtils.java
index 3c2cf3f2..dde35b27 100644
--- a/src/main/java/org/apache/commons/lang3/StringUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringUtils.java
@@ -3227,6 +3227,18 @@ public class StringUtils {
      * @since 3.0 Changed signature to use varargs
      */
     public static <T> String join(T... elements) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                return join_original(elements);
+            } catch (NullPointerException e) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+        } else {
+            return join_original(elements);
+        }
+    }
+
+    public static <T> String join_original(T... elements) {
         return join(elements, null);
     }
```

</details>

<details>
<summary><b>Lang-22</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-662
* new tag: `D4J_Lang_22_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/Fraction.java b/src/main/java/org/apache/commons/lang3/math/Fraction.java
index b36a156a..b2dc787c 100644
--- a/src/main/java/org/apache/commons/lang3/math/Fraction.java
+++ b/src/main/java/org/apache/commons/lang3/math/Fraction.java
@@ -579,6 +579,19 @@ public final class Fraction extends Number implements Comparable<Fraction> {
      * @return the greatest common divisor, never zero
      */
     private static int greatestCommonDivisor(int u, int v) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            int returnValue;
+            returnValue = greatestCommonDivisor_original(u, v);
+            if (u == Integer.MIN_VALUE && v == 2 && returnValue != 2) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+            return returnValue;
+        } else {
+            return greatestCommonDivisor_original(u, v);
+        }
+    }
+
+    private static int greatestCommonDivisor_original(int u, int v) {
         // From Commons Math:
         //if either operand is abs 1, return 1:
         if (Math.abs(u) <= 1 || Math.abs(v) <= 1) {
```

</details>

<details>
<summary><b>Lang-35</b> (ARJA correct)</summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-571
* new tag: `D4J_Lang_35_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/ArrayUtils.java b/src/main/java/org/apache/commons/lang3/ArrayUtils.java
index ac22f8fd..32d6f0e5 100644
--- a/src/main/java/org/apache/commons/lang3/ArrayUtils.java
+++ b/src/main/java/org/apache/commons/lang3/ArrayUtils.java
@@ -3286,6 +3286,22 @@ public class ArrayUtils {
      * @throws IllegalArgumentException if both arguments are null
      */
     public static <T> T[] add(T[] array, T element) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                return add_original(array, element);
+            } catch (ClassCastException e) {
+                if (array == null && element == null) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+        } else {
+            return add_original(array, element);
+        }
+    }
+
+    public static <T> T[] add_original(T[] array, T element) {
         Class<?> type;
         if (array != null){
             type = array.getClass();
@@ -3565,6 +3581,22 @@ public class ArrayUtils {
      * @throws IllegalArgumentException if both array and element are null
      */
     public static <T> T[] add(T[] array, int index, T element) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                return add_original(array, index, element);
+            } catch (ClassCastException e) {
+                if (array == null && element == null) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+        } else {
+            return add_original(array, index, element);
+        }
+    }
+
+    public static <T> T[] add_original(T[] array, int index, T element) {
         Class<?> clss = null;
         if (array != null) {
             clss = array.getClass().getComponentType();
```

</details>

<details>
<summary><b>Math-2</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-1021
* new tag: `D4J_Math_2_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java b/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
index 27691272f..59287e081 100644
--- a/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
+++ b/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
@@ -110,6 +110,19 @@ public class HypergeometricDistribution extends AbstractIntegerDistribution {
         this.sampleSize = sampleSize;
     }

+    @Override
+    public int sample() {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            int returnValue = super.sample();
+            if (returnValue < this.getSupportLowerBound() || returnValue > this.getSupportUpperBound()) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+            return returnValue;
+        } else {
+            return super.sample();
+        }
+    }
+
     /** {@inheritDoc} */
     public double cumulativeProbability(int x) {
         double ret;
```
</details>

<details>
<summary><b>Math-8</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-942
* new tag: `D4J_Math_8_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math3/distribution/DiscreteDistribution.java b/src/main/java/org/apache/commons/math3/distribution/DiscreteDistribution.java
index 5cb0e4382..58c8b5b54 100644
--- a/src/main/java/org/apache/commons/math3/distribution/DiscreteDistribution.java
+++ b/src/main/java/org/apache/commons/math3/distribution/DiscreteDistribution.java
@@ -179,6 +179,26 @@ public class DiscreteDistribution<T> {
      * positive.
      */
     public T[] sample(int sampleSize) throws NotStrictlyPositiveException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            T[] resultValue = null;
+            try {
+                resultValue = sample_orig(sampleSize);
+            } catch (ArrayStoreException e) {
+                Class typeT = ((T) new Object()).getClass();
+                Object singletonObject = singletons.get(0);
+                if (typeT.isInstance(singletonObject) && !typeT.equals(singletonObject.getClass())) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+            return resultValue;
+        } else {
+            return sample_orig(sampleSize);
+        }
+    }
+
+    public T[] sample_orig(int sampleSize) throws NotStrictlyPositiveException {
         if (sampleSize <= 0) {
             throw new NotStrictlyPositiveException(LocalizedFormats.NUMBER_OF_SAMPLES,
                     sampleSize);
```
</details>



## Not Supported Subjects

The following list includes subjects, for which the bug report does not contain sufficient information to formulate a meaningful assertion. **Total count: 1** subject.

<details>
<summary><b>Math-6</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-949

> "The method LevenbergMarquardtOptimizer.getIterations() does not report the correct number of iterations; It always returns 0. A quick look at the code shows that only SimplexOptimizer calls BaseOptimizer.incrementEvaluationsCount()
> 
> I've put a test case below. Notice how the evaluations count is correctly incremented, but the iterations count is not."

The bug report says that the method always returns zero but does not say when it is correct and when it is incorrect. It provides a test case; however, this one is already included in the defects4j test suite.

</details>
