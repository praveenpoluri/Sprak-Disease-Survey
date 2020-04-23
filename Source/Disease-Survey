spark.conf.set("spark.sql.shuffle.partitions", 2)
    val df = spark.read.format("csv").option("header","true").option("inferSchema","true") .option("nullValue","NA").option("timestampFormat","yyyy-MM-dd'T'HH:mm?:ss").option("mode","failfast").option("path","survey.csv").load()
    val df1 = df.select( $"Gender",$"treatment")
    val df2 = df.select($"Gender",
                         (when($"treatment" === "Yes", 1).otherwise(0)).alias("All-Yes"),
                         (when($"treatment" === "No", 1).otherwise(0)).alias("All-Nos")
                       )
    def parseGender(g: String) = {  
      g.toLowerCase match {
        case "male" | "m" | "male-ish" | "maile" |
             "mal" | "male (cis)" | "make" | "male " |
             "man" | "msle" | "mail" | "malr" |
             "cis man" | "cis male" => "Male"
        case "cis female" | "f" | "female" |
             "woman" |  "femake" | "female " |
             "cis-female/femme" | "female (cis)" |
             "femail" => "Female"
        case _ => "Transgender"
       }
       }
    val parseGenderUDF = udf(parseGender _)
    val df3 = df2.select((parseGenderUDF($"Gender")).alias("Gender"),
                          $"All-Yes",
                          $"All-Nos"
                        )
    val df4 = df3.groupBy("Gender").agg( sum($"All-Yes"),sum($"All-Nos"))
    val df5 = df4.filter($"Gender" =!= "Transgender")
    df5.collect                                           