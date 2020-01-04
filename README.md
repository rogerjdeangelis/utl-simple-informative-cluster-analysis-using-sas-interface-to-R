# utl-simple-informative-cluster-analysis-using-sas-interface-to-R
Simple informative cluster analysis using sas interface with R
    Simple informative cluster analysis using sas interface with R

    Graph output
    https://tinyurl.com/yx3ux6sa
    https://github.com/rogerjdeangelis/utl-simple-informative-cluster-analysis-using-sas-interface-to-R/blob/master/cluster.pdf

    github
    https://tinyurl.com/tmfpyg2
    https://github.com/rogerjdeangelis/utl-simple-informative-cluster-analysis-using-sas-interface-to-R

    Stackoverflow
    https://tinyurl.com/yxxcmuyc
    https://stackoverflow.com/questions/59541292/r-finding-elements-matching-with-each-other-within-a-vector

    Jay Sf profiile
    https://stackoverflow.com/users/6574038/jay-sf

    SOAPBOX ON;

      It seems to me that R and Python are legends in thier own minds.

      Possible downfall of R and Python as languages, but not their packages?

      With dozens of data structures and datatypes with hundreds of functions unique to each data structure or
      or data type, commumication with the other languares is extremely problematic.

      I was UNABLE to convert a two level factor structure to a data frame after about an hour of trying.

      This happens all the time.

      The beauty of SAS lies in just one data structure and just two data types.
      It is remarkable what SAS can do with a 64bit float, ie float to IBM packed decimal for instance.
      Would like to see a 128bit float, we have 3,4,5,6,7 and 8 byte floats.

      Less is often more?

    SOAPBOX OFF

    *_                   _
    (_)_ __  _ __  _   _| |_
    | | '_ \| '_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
     informat adr $44.;
     input adr &;
    cards4;
    wall street
    Wall-street
    Wall ST
    andheri pump house
    weh, nr. pump house
    Wallstreet
    weh andheri pump house
    Wall Street
    weh andheri pump house et
    andheri at weh pump house
    andheri pump house(mt)
    ;;;;
    run;quit;

     WORK.HAVE total obs=11

       ADR

       wall street
       Wall-street
       Wall ST
       andheri pump house
       weh, nr. pump house
       Wallstreet
       weh andheri pump house
       Wall Street
       weh andheri pump house et
       andheri at weh pump house
       andheri pump house(mt)

    *            _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| '_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|


    MAIN Branch of cluster output (binary split);

     WORK.WANT total obs=11

       TEXT                         LABEL

       Wallstreet                   Wall Street
       Wall-street                  Wall Street
       Wall ST                      Wall Street
       wall street                  Wall Street
       Wall Street                  Wall Street

       weh andheri pump house       Andheri Pump House
       andheri pump house(mt)       Andheri Pump House
       weh, nr. pump house          Andheri Pump House
       andheri at weh pump house    Andheri Pump House
       andheri pump house           Andheri Pump House
       weh andheri pump house et    Andheri Pump House


    Not the graphical solutuion

       ___________________________________________
       |                  |                      |
                          |                      |
     Wall ST              |                      |
            +----------------------+             |
            |           |          |             |
       Wallstreet Wall-street      |             |
                           +-----------+         |
                           |           |         |
                       wall street  wall street  |
                                                 |
                                                 |
                                   +---------------------+
                                   |                     |
                          weh, nr. pump house            |
                                              +---------------------------+
                                              |                           |
                                              |                           |
                               weh andheri pump house           weh andheri pump house et
                                          |                                      |
                                          |                                      |
                   +--------------------------------+                  +-------------------------------+
                   |                                |                  |                               |
        weh andheri pump house        weh andheri pump house et    weh, nr. pump house    andheri pump house

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;


    proc datasets lib=work;
     delete labels values want;
    run;quit;

    %utlfkil(d:/xpt/want.xpt);

    %utl_submit_r64('
    library(SASxport);
    library(data.table);
    x <- (c("wall street", "Wall-street", "Wall ST", "andheri pump house",
      "weh, nr. pump house", "Wallstreet", "weh andheri pump house",
      "Wall Street", "weh andheri pump house et", "andheri at weh pump house",
      "andheri pump house(mt)"));
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    x<-have$ADR;
    y<-x;
    e  <- adist(na.omit(tolower(x)));
    rownames(e) <- na.omit(x);
    hc <- hclust(as.dist(e));
    pdf("d:/pdf/cluster.pdf");
    plot(hc);
    smly <- cutree(hc, h=16);
    key <- data.table(x=na.omit(x),
      smly=factor(smly, labels=c("Wall Street", "Andheri Pump House")),
      row.names=NULL);
    x <- key$smly;
    labels<-as.data.table(levels(x));
    colnames(labels)="LABEL";
    values<-as.data.table(cbind(y,factor(x)));
    colnames(values)<-c("TEXT","KEY");
    write.xport(labels,values,file="d:/xpt/want.xpt");
    ');

    libname xpt xport "d:/xpt/want.xpt";
    data labels;
      set xpt.labels;
      key=_n_;
    run;quit;
    data values;
      set xpt.values;
    run;quit;
    proc sql;
      create
          table want as
      select
          l.text
         ,r.label
      from
          values as l, labels as r
      where
          input(l.key,3.) = r.key
      order
          by l.key
    ;quit;
    libname xpt clear;


