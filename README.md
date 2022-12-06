# defects4j-instrumented

This repository includes the instrumented defects4j subjects. We manually transformed the information from the bug report into an oracle, which is added to the buggy version of each subject.

Each modification is tagged as: `D4J_<project>_<bugID>_BUGGY_VERSION_INSTRUMENTED`

The zipped git archives are included in [instrumented-archives](./instrumented-archives).

*Note: For now, we will only have direct modifications of the buggy source code. Later, we want to introduce an automated mechanism that (given the source code modification) directly instruments the Java bytecode.*

## Current general issues

* not all test cases call the method mentioned in the bug report; they call the underlying buggy method
* with the additional specificaton from the bug report, other test cases can also fail. They should be fixed with the correct bug though. E.g., Lang-7, a test case interpretes both (the incorrect and the correct) behavior a suitable way so that the test cases passes; but with our instrumentation it fails because we throw a RuntimeException. Correct: throw NumberFormatException. Buggy: return null. The other test case handles both but not our RuntimeException.

## Methodology

* we start with subjects, for which ARJA can generated plausible patch
* ARJA
	* considers 224 bugs from Defects4J
	* plausible patches: 59
	* correct patches: 18

## Example: Lang-19

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
index d84aae58..2a0d1f3d 100644
--- a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
@@ -465,7 +465,17 @@ public class StringEscapeUtils {
      * @since 3.0
      */
     public static final String unescapeHtml4(String input) {
-        return UNESCAPE_HTML4.translate(input);
+        try {
+            // Original Code START
+            return UNESCAPE_HTML4.translate(input);
+            // Original Code END
+        } catch (StringIndexOutOfBoundsException e) {
+            if (input.contains("&")) {
+                throw new RuntimeException("Execution violates behavior specified in the bug report.");
+            } else {
+                throw e;
+            }
+        }
     }
```

## Progress

The following list includes the already covered subjects. **Total count: 6** subjects, for which ARJA can generate a plausible patch.

<details>
<summary><b>Lang-19 (Example; cannot be fixed by ARJA)</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-710
* new tag: `D4J_Lang_19_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
index d84aae58..2a0d1f3d 100644
--- a/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringEscapeUtils.java
@@ -465,7 +465,17 @@ public class StringEscapeUtils {
      * @since 3.0
      */
     public static final String unescapeHtml4(String input) {
-        return UNESCAPE_HTML4.translate(input);
+        try {
+            // Original Code START
+            return UNESCAPE_HTML4.translate(input);
+            // Original Code END
+        } catch (StringIndexOutOfBoundsException e) {
+            if (input.contains("&")) {
+                throw new RuntimeException("Execution violates behavior specified in the bug report.");
+            } else {
+                throw e;
+            }
+        }
     }
```

</details>

<details>
<summary><b>Lang-7</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-822
* new tag: `D4J_Lang_7_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
index d49da7f4..73e1e67e 100644
--- a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
+++ b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
@@ -443,7 +443,20 @@ public class NumberUtils {
      * @throws NumberFormatException if the value cannot be converted
      */
     public static Number createNumber(String str) throws NumberFormatException {
-        if (str == null) {
+       Number returnValue = null;
+       try {
+               returnValue = createNumber_original(str);
+       } catch (NumberFormatException e) {
+               throw e;
+       }
+       if (str != null && str.startsWith("--") && returnValue == null) {
+               throw new RuntimeException("Execution violates behavior specified in the bug report.");
+       }
+       return returnValue;
+    }
+
+    public static Number createNumber_original(String str) throws NumberFormatException {
+       if (str == null) {
             return null;
         }
         if (StringUtils.isBlank(str)) {
```

</details>

<details>
<summary><b>Lang-16</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-746
* new tag: `D4J_Lang_16_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
index 882358f2..4b40ded4 100644
--- a/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
+++ b/src/main/java/org/apache/commons/lang3/math/NumberUtils.java
@@ -442,6 +442,23 @@ public class NumberUtils {
      * @throws NumberFormatException if the value cannot be converted
      */
     public static Number createNumber(String str) throws NumberFormatException {
+       try {
+               return createNumber_original(str);
+       } catch (NumberFormatException e) {
+               if (str != null && (str.startsWith("0X") || str.startsWith("-0X"))) {
+                       try {
+                               Integer.decode(str);
+                       } catch (NumberFormatException e_decode) {
+                               throw e;
+                       }
+                       throw new RuntimeException("Execution violates behavior specified in the bug report.");
+               } else {
+                       throw e;
+               }
+       }
+    }
+
+    public static Number createNumber_original(String str) throws NumberFormatException {
         if (str == null) {
             return null;
         }
```

</details>

<details>
<summary><b>Lang-20</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-703
* new tag: `D4J_Lang_20_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/StringUtils.java b/src/main/java/org/apache/commons/lang3/StringUtils.java
index 3c2cf3f2..512110b9 100644
--- a/src/main/java/org/apache/commons/lang3/StringUtils.java
+++ b/src/main/java/org/apache/commons/lang3/StringUtils.java
@@ -3227,6 +3227,14 @@ public class StringUtils {
      * @since 3.0 Changed signature to use varargs
      */
     public static <T> String join(T... elements) {
+       try {
+               return join_original(elements);
+       } catch (NullPointerException e) {
+               throw new RuntimeException("Execution violates behavior specified in the bug report.");
+       }
+    }
+
+    public static <T> String join_original(T... elements) {
         return join(elements, null);
     }
```

</details>

<details>
<summary><b>Lang-22</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-662
* new tag: `D4J_Lang_22_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/math/Fraction.java b/src/main/java/org/apache/commons/lang3/math/Fraction.java
index b36a156a..c7020af7 100644
--- a/src/main/java/org/apache/commons/lang3/math/Fraction.java
+++ b/src/main/java/org/apache/commons/lang3/math/Fraction.java
@@ -579,6 +579,15 @@ public final class Fraction extends Number implements Comparable<Fraction> {
      * @return the greatest common divisor, never zero
      */
     private static int greatestCommonDivisor(int u, int v) {
+       int returnValue;
+       returnValue = greatestCommonDivisor_original(u, v);
+       if (u == Integer.MIN_VALUE && v == 2 && returnValue != 2) {
+               throw new RuntimeException("Execution violates behavior specified in the bug report.");
+       }
+       return returnValue;
+    }
+
+    private static int greatestCommonDivisor_original(int u, int v) {
         // From Commons Math:
         //if either operand is abs 1, return 1:
         if (Math.abs(u) <= 1 || Math.abs(v) <= 1) {
```

</details>

<details>
<summary><b>Lang-35</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/LANG-571
* new tag: `D4J_Lang_35_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/lang3/ArrayUtils.java b/src/main/java/org/apache/commons/lang3/ArrayUtils.java
index ac22f8fd..cbdb0239 100644
--- a/src/main/java/org/apache/commons/lang3/ArrayUtils.java
+++ b/src/main/java/org/apache/commons/lang3/ArrayUtils.java
@@ -3286,6 +3286,18 @@ public class ArrayUtils {
      * @throws IllegalArgumentException if both arguments are null
      */
     public static <T> T[] add(T[] array, T element) {
+       try {
+               return add_original(array, element);
+       } catch (ClassCastException e) {
+               if (array == null && element == null) {
+                       throw new RuntimeException("Execution violates behavior specified in the bug report.");
+               } else {
+                       throw e;
+               }
+       }
+    }
+
+    public static <T> T[] add_original(T[] array, T element) {
         Class<?> type;
         if (array != null){
             type = array.getClass();
@@ -3565,6 +3577,18 @@ public class ArrayUtils {
      * @throws IllegalArgumentException if both array and element are null
      */
     public static <T> T[] add(T[] array, int index, T element) {
+       try {
+               return add_original(array, index, element);
+       } catch (ClassCastException e) {
+               if (array == null && element == null) {
+                       throw new RuntimeException("Execution violates behavior specified in the bug report.");
+               } else {
+                       throw e;
+               }
+       }
+    }
+
+    public static <T> T[] add_original(T[] array, int index, T element) {
         Class<?> clss = null;
         if (array != null) {
             clss = array.getClass().getComponentType();
```

</details>

<details>
<summary><b>Math-2</b></summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-1021
* new tag: `D4J_Math_2_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java b/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
index 27691272f..2c76b29fe 100644
--- a/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
+++ b/src/main/java/org/apache/commons/math3/distribution/HypergeometricDistribution.java
@@ -109,6 +109,15 @@ public class HypergeometricDistribution extends AbstractIntegerDistribution {
         this.populationSize = populationSize;
         this.sampleSize = sampleSize;
     }
+
+    @Override
+    public int sample() {
+       int returnValue = super.sample();
+       if (returnValue < this.getSupportLowerBound() || returnValue > this.getSupportUpperBound()) {
+               throw new RuntimeException("Execution violates behavior specified in the bug report.");
+       }
+       return returnValue;
+    }

     /** {@inheritDoc} */
     public double cumulativeProbability(int x) {
```

</details>