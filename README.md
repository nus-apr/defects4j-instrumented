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

The following list includes the already covered subjects. **Total count: 17** subjects.

* 2 ARJA cannot produce a plausible patch
* 13 ARJA can generate a plausible but incorrect patch
* 2 ARJA can produce correct patch.


<details>
<summary><b>Lang-19</b> (Example; ARJA implausible)</summary>

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

<details>
<summary><b>Math-20</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-864
* new tag: `D4J_Math_20_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math3/optimization/direct/CMAESOptimizer.java b/src/main/java/org/apache/commons/math3/optimization/direct/CMAESOptimizer.java
index 4b7dbf6bb..7465d02ea 100644
--- a/src/main/java/org/apache/commons/math3/optimization/direct/CMAESOptimizer.java
+++ b/src/main/java/org/apache/commons/math3/optimization/direct/CMAESOptimizer.java
@@ -316,6 +316,21 @@ public class CMAESOptimizer
         this.generateStatistics = generateStatistics;
     }

+    @Override
+    public PointValuePair optimize(int maxEval, MultivariateFunction f, GoalType goalType, double[] startPoint,
+            double[] lower, double[] upper) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            PointValuePair resultValue = super.optimize(maxEval, f, goalType, startPoint, lower, upper);
+            if (resultValue.getPoint()[0] > upper[0]) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            } else {
+                return resultValue;
+            }
+        } else {
+            return super.optimize(maxEval, f, goalType, startPoint, lower, upper);
+        }
+    }
+
     /**
      * @return History of sigma values.
      */
```
</details>

<details>
<summary><b>Math-31</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-718
* new tag: `D4J_Math_31_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math3/distribution/BinomialDistribution.java b/src/main/java/org/apache/commons/math3/distribution/BinomialDistribution.java
index 505b93f3b..a761347fc 100644
--- a/src/main/java/org/apache/commons/math3/distribution/BinomialDistribution.java
+++ b/src/main/java/org/apache/commons/math3/distribution/BinomialDistribution.java
@@ -18,6 +18,7 @@ package org.apache.commons.math3.distribution;

 import org.apache.commons.math3.exception.OutOfRangeException;
 import org.apache.commons.math3.exception.NotPositiveException;
+import org.apache.commons.math3.exception.NumberIsTooLargeException;
 import org.apache.commons.math3.exception.util.LocalizedFormats;
 import org.apache.commons.math3.special.Beta;
 import org.apache.commons.math3.util.FastMath;
@@ -59,6 +60,22 @@ public class BinomialDistribution extends AbstractIntegerDistribution {
         numberOfTrials = trials;
     }

+    @Override
+    public int inverseCumulativeProbability(final double p) throws OutOfRangeException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            int resultValue = super.inverseCumulativeProbability(p);
+            if (p == 0.5 && this.probabilityOfSuccess == 0.5) {
+                int expectedValue = (int) (numberOfTrials * 0.5);
+                if (resultValue < expectedValue - 1 || resultValue > expectedValue + 1) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                }
+            }
+            return resultValue;
+        } else {
+            return super.inverseCumulativeProbability(p);
+        }
+    }
+
     /**
      * Access the number of trials for this distribution.
      *
```
</details>

<details>
<summary><b>Math-49</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-645
* new tag: `D4J_Math_49_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/linear/OpenMapRealVector.java b/src/main/java/org/apache/commons/math/linear/OpenMapRealVector.java
index 5db488466..19656d939 100644
--- a/src/main/java/org/apache/commons/math/linear/OpenMapRealVector.java
+++ b/src/main/java/org/apache/commons/math/linear/OpenMapRealVector.java
@@ -17,6 +17,7 @@
 package org.apache.commons.math.linear;

 import java.io.Serializable;
+import java.util.ConcurrentModificationException;

 import org.apache.commons.math.exception.MathArithmeticException;
 import org.apache.commons.math.exception.util.LocalizedFormats;
@@ -365,6 +366,20 @@ public class OpenMapRealVector extends AbstractRealVector

     /** {@inheritDoc} */
     public OpenMapRealVector ebeMultiply(RealVector v) {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            OpenMapRealVector returnValue = null;
+            try {
+                returnValue = ebeMultiply_original(v);
+            } catch (ConcurrentModificationException e) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+            return returnValue;
+        } else {
+            return ebeMultiply_original(v);
+        }
+    }
+
+    public OpenMapRealVector ebeMultiply_original(RealVector v) {
         checkVectorDimensions(v.getDimension());
         OpenMapRealVector res = new OpenMapRealVector(this);
         Iterator iter = res.entries.iterator();
```
</details>

<details>
<summary><b>Math-56</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-552
* new tag: `D4J_Math_56_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/util/MultidimensionalCounter.java b/src/main/java/org/apache/commons/math/util/MultidimensionalCounter.java
index 56c9ffebc..efb2647af 100644
--- a/src/main/java/org/apache/commons/math/util/MultidimensionalCounter.java
+++ b/src/main/java/org/apache/commons/math/util/MultidimensionalCounter.java
@@ -242,6 +242,12 @@ public class MultidimensionalCounter implements Iterable<Integer> {
         --idx;
         indices[last] = idx;

+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            if (indices[last] != index - count) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+        }
+
         return indices;
     }
```
</details>

<details>
<summary><b>Math-60</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-414
* new tag: `D4J_Math_60_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/distribution/NormalDistributionImpl.java b/src/main/java/org/apache/commons/math/distribution/NormalDistributionImpl.java
index 0e124d852..f9649d5e5 100644
--- a/src/main/java/org/apache/commons/math/distribution/NormalDistributionImpl.java
+++ b/src/main/java/org/apache/commons/math/distribution/NormalDistributionImpl.java
@@ -122,6 +122,22 @@ public class NormalDistributionImpl extends AbstractContinuousDistribution
      * @throws MathException if the algorithm fails to converge
      */
     public double cumulativeProbability(double x) throws MathException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                return cumulativeProbability_original(x);
+            } catch (org.apache.commons.math.ConvergenceException e) {
+                if (x < (mean - 20 * standardDeviation) || x > (mean + 20 * standardDeviation)) {
+                    throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                } else {
+                    throw e;
+                }
+            }
+        } else {
+            return cumulativeProbability_original(x);
+        }
+    }
+
+    public double cumulativeProbability_original(double x) throws MathException {
         final double dev = x - mean;
         try {
         return 0.5 * (1.0 + Erf.erf((dev) /
```
</details>

<details>
<summary><b>Math-65</b> (ARJA implausible)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-377
* new tag: `D4J_Math_65_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/optimization/general/AbstractLeastSquaresOptimizer.java b/src/main/java/org/apache/commons/math/optimization/general/AbstractLeastSquaresOptimizer.java
index 30ebfff21..6f91c53a3 100644
--- a/src/main/java/org/apache/commons/math/optimization/general/AbstractLeastSquaresOptimizer.java
+++ b/src/main/java/org/apache/commons/math/optimization/general/AbstractLeastSquaresOptimizer.java
@@ -252,6 +252,19 @@ public abstract class AbstractLeastSquaresOptimizer implements DifferentiableMul
      * @return chi-square value
      */
     public double getChiSquare() {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            double returnValue = getChiSquare_original();
+            if (getRMS() != Math.sqrt(returnValue / rows)) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            } else {
+                return returnValue;
+            }
+        } else {
+            return getChiSquare_original();
+        }
+    }
+
+    public double getChiSquare_original() {
         double chiSquare = 0;
         for (int i = 0; i < rows; ++i) {
             final double residual = residuals[i];
```
</details>

<details>
<summary><b>Math-71</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-358
* new tag: `D4J_Math_71_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/ode/nonstiff/DormandPrince853Integrator.java b/src/main/java/org/apache/commons/math/ode/nonstiff/DormandPrince853Integrator.java
index 2d6b17e29..0198886ec 100644
--- a/src/main/java/org/apache/commons/math/ode/nonstiff/DormandPrince853Integrator.java
+++ b/src/main/java/org/apache/commons/math/ode/nonstiff/DormandPrince853Integrator.java
@@ -17,6 +17,9 @@

 package org.apache.commons.math.ode.nonstiff;

+import org.apache.commons.math.ode.DerivativeException;
+import org.apache.commons.math.ode.FirstOrderDifferentialEquations;
+import org.apache.commons.math.ode.IntegratorException;

 /**
  * This class implements the 8(5,3) Dormand-Prince integrator for Ordinary
@@ -235,6 +238,22 @@ public class DormandPrince853Integrator extends EmbeddedRungeKuttaIntegrator {
           minStep, maxStep, vecAbsoluteTolerance, vecRelativeTolerance);
   }

+  @Override
+  public double integrate(final FirstOrderDifferentialEquations equations,
+                          final double t0, final double[] y0,
+                          final double t, final double[] y)
+  throws DerivativeException, IntegratorException {
+      if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+          double finalT = super.integrate(equations, t0, y0, t, y);
+          if (finalT < t - 1.0e-6 || finalT > t + 1.0e-6) {
+              throw new RuntimeException("[Defects4J_BugReport_Violation]");
+          }
+          return finalT;
+      } else {
+          return super.integrate(equations, t0, y0, t, y);
+      }
+  }
+
   /** {@inheritDoc} */
   @Override
   public int getOrder() {
```
</details>

<details>
<summary><b>Math-74</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-338
* new tag: `D4J_Math_71_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/ode/nonstiff/EmbeddedRungeKuttaIntegrator.java b/src/main/java/org/apache/commons/math/ode/nonstiff/EmbeddedRungeKuttaIntegrator.java
index 6f3e88358..d8b83d2c7 100644
--- a/src/main/java/org/apache/commons/math/ode/nonstiff/EmbeddedRungeKuttaIntegrator.java
+++ b/src/main/java/org/apache/commons/math/ode/nonstiff/EmbeddedRungeKuttaIntegrator.java
@@ -246,8 +246,26 @@ public abstract class EmbeddedRungeKuttaIntegrator
           if (vecAbsoluteTolerance == null) {
               scale = new double[y0.length];
               java.util.Arrays.fill(scale, scalAbsoluteTolerance);
+
+              if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+                  for (int i=0; i<scale.length; i++) {
+                      double yi = Math.max(Math.abs(y0[i]), Math.abs(y0[i]));
+                      if (scale[i] != scalAbsoluteTolerance + scalRelativeTolerance * yi) {
+                          throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                      }
+                  }
+              }
             } else {
               scale = vecAbsoluteTolerance;
+
+              if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+                  for (int i=0; i<scale.length; i++) {
+                      double yi = Math.max(Math.abs(y0[i]), Math.abs(y0[i]));
+                      if (scale[i] != vecAbsoluteTolerance[i] + vecAbsoluteTolerance[i] * yi) {
+                          throw new RuntimeException("[Defects4J_BugReport_Violation]");
+                      }
+                  }
+              }
             }
           hNew = initializeStep(equations, forward, getOrder(), scale,
                                 stepStart, y, yDotK[0], yTmp, yDotK[1]);
```
</details>

<details>
<summary><b>Math-81</b> (ARJA plausible but incorrect)</summary>

* Bug Report: https://issues.apache.org/jira/browse/MATH-308
* new tag: `D4J_Math_81_BUGGY_VERSION_INSTRUMENTED`

```diff
diff --git a/src/main/java/org/apache/commons/math/linear/EigenDecompositionImpl.java b/src/main/java/org/apache/commons/math/linear/EigenDecompositionImpl.java
index 2d0d72f22..2c95c46ab 100644
--- a/src/main/java/org/apache/commons/math/linear/EigenDecompositionImpl.java
+++ b/src/main/java/org/apache/commons/math/linear/EigenDecompositionImpl.java
@@ -186,10 +186,21 @@ public class EigenDecompositionImpl implements EigenDecomposition {
      * @exception InvalidMatrixException (wrapping a {@link
      * org.apache.commons.math.ConvergenceException} if algorithm fails to converge
      */
-    public EigenDecompositionImpl(final double[] main, double[] secondary,
-            final double splitTolerance)
-        throws InvalidMatrixException {
+    public EigenDecompositionImpl(final double[] main, double[] secondary, final double splitTolerance)
+            throws InvalidMatrixException {
+        if (Boolean.valueOf(System.getProperty("defects4j.instrumentation.enabled"))) {
+            try {
+                this.init_original(main, secondary, splitTolerance);
+            } catch (ArrayIndexOutOfBoundsException e) {
+                throw new RuntimeException("[Defects4J_BugReport_Violation]");
+            }
+        } else {
+            this.init_original(main, secondary, splitTolerance);
+        }
+    }

+    private void init_original(final double[] main, double[] secondary, final double splitTolerance)
+            throws InvalidMatrixException {
         this.main      = main.clone();
         this.secondary = secondary.clone();
         transformer    = null;
```
</details>


## Not Supported Subjects

The following list includes subjects, for which the bug report does not contain sufficient information to formulate a meaningful assertion. **Total count: 4** subject.

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


