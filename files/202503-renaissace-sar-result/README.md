# Environment

- OpenJDK: latest. commit `037e47112bd`
- Renaissance: 0.16 jmh version
- test command to evaluation:

```bash
opts="-Xms8g -Xmx8g -XX:+AlwaysPreTouch -XX:ObjectAlignmentInBytes=64"
opts+=" --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/sun.nio.ch=ALL-UNNAMED"
opts+=" --add-opens=java.base/java.nio=ALL-UNNAMED --add-opens=java.base/java.lang.invoke=ALL-UNNAMED"
opts+=" --add-opens=jdk.compiler/com.sun.tools.javac=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED"
opts+=" --add-opens=java.base/java.lang.reflect=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED"
opts+=" --add-exports=java.base/jdk.internal.ref=ALL-UNNAMED --add-exports=jdk.unsupported/sun.misc=ALL-UNNAMED"
opts+=" --add-exports=jdk.compiler/com.sun.tools.javac.file=ALL-UNNAMED"
java -Xmx2g -Djmh.blackhole.autoDetect=true -jar renaissance-jmh-0.16.0.jar -f 3 -i 10 -wi 10 -jvmArgsAppend "$opts" ScalaKmeans
```

# Execution time for each benchmark

```text
0:01:01	org.renaissance.actors.JmhAkkaUct.run
0:07:26	org.renaissance.actors.JmhReactors.run
0:01:39	org.renaissance.apache.spark.JmhAls.run
0:00:37	org.renaissance.apache.spark.JmhChiSquare.run
0:00:55	org.renaissance.apache.spark.JmhDecTree.run
0:03:56	org.renaissance.apache.spark.JmhGaussMix.run
0:01:02	org.renaissance.apache.spark.JmhLogRegression.run
0:06:17	org.renaissance.apache.spark.JmhMovieLens.run
0:00:50	org.renaissance.apache.spark.JmhNaiveBayes.run
0:04:32	org.renaissance.apache.spark.JmhPageRank.run
0:04:56	org.renaissance.database.JmhDbShootout.run
0:01:10	org.renaissance.jdk.concurrent.JmhFjKmeans.run
0:01:06	org.renaissance.jdk.concurrent.JmhFutureGenetic.run
0:02:09	org.renaissance.jdk.streams.JmhMnemonics.run
0:01:51	org.renaissance.jdk.streams.JmhParMnemonics.run
0:00:09	org.renaissance.jdk.streams.JmhScrabble.run
0:01:48	org.renaissance.neo4j.JmhNeo4jAnalytics.run
0:00:10	org.renaissance.rx.JmhRxScrabble.run
0:01:37	org.renaissance.scala.dotty.JmhDotty.run
0:01:35	org.renaissance.scala.sat.JmhScalaDoku.run
0:00:14	org.renaissance.scala.stdlib.JmhScalaKmeans.run
0:03:44	org.renaissance.scala.stm.JmhPhilosophers.run
0:00:57	org.renaissance.scala.stm.JmhScalaStmBench7.run
0:03:33	org.renaissance.twitter.finagle.JmhFinagleChirper.run
0:03:10	org.renaissance.twitter.finagle.JmhFinagleHttp.run
```

Note that: benchmark `FinagleChirper` has exception.
