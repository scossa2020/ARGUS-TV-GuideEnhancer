#ARGUS-TV-GuideEnhancer
**Electronic Program Guide Enhancer for ARGUS TV 2.3.0+**

[ARGUS TV](http://http://www.argus-tv.com/ "ARGUS TV") is a sophisticated scheduling/recording engine. It has it's own Electronic Program Guide (EPG). You can populate the guide from many sources (i.e., xmltv, SchedulesDirect, etc.).

Regardless of the method with which you populate the EPG, you may want to "enrich" your guide data to add Season and Episode numbers (SxxExx). This "enrich"-ing is useful for [XBMC/Kodi](http://www.kodi.tv "Kodi") or [MediaPortal](http://www.team-mediaportal.com/ "") plugin TVSeries that automatically download episode information, backdrops, ratings, etc. By enriching your guide data, you can automatically have the SxxExx information in your file names and thus have TVSeries register your recordings with no manual intervention.

##Getting Started
If you are going to use the Guide Enricher, follow these steps.

###Installation
- .NET Framework 4.5 is required (web installer http://go.microsoft.com/fwlink/p/?LinkId=397703 or offline installer http://go.microsoft.com/fwlink/p/?LinkId=397706)
- Download the [Guide Enricher installer](https://github.com/ChrisRichner/ARGUS-TV-GuideEnhancer/releases "Guide Enricher Releases")
- Make sure you have ARGUS-TV 2.3.0+ installed and running
- Run the Guide Enricher installer
- Only option is path where to install it...
- After a successful install (hopefully), you will have a windows service called GuideEnricher available

###Configuration
You can customize the *GuideEnricherService.exe.config* file in the program directory.

#####Application Settings
```xml
<appSettings>
    <!-- Location to store cache of thetvdb.com data -->
    <add key="TvDbLibCache" value="c:\\tvdblibcache\\"/>
    <!-- uncomment the following line and set it to your language if you want to use another language than en (de for German, fr for French, ...) -->
    <add key="TvDbLanguage" value="de"/>
    <!-- 
         how long the wait thread waits before doing a run of guide enricher 
         
       -->
    <add key="sleepTimeInHours" value="12"/>

    <!--
         This number determines the maxiumum number of shows that can be updated at once.
         If more than this need to be updated, the GuideEnricher will loop through 
         the shows by maxShowNumberPerUpdate until they are all updated.

         The reason to limit the number of shows is due to the FTR server timing out
         the webservice call for taking too long to update the shows.  So the number is
         probably dependent on the FTR server performance
      -->
    <add key="maxShowNumberPerUpdate" value="20"/>

    <!--
         The next properties are used to build the
         URL to the FTR server
      -->
    <add key="ftrUrlHost" value="localhost"/>
    <add key="ftrUrlPort" value="49943"/>
    <!-- leave ftrUrlPassword blank if you don't use password -->
    <add key="ftrUserName" value=""/>
    <add key="ftrUrlPassword" value=""/>
    <!-- 
      set dumpepisodes to 'true' if you want all episodes for a series dumped in the log file
      make sure logging is set to at least info level
    -->
    <add key="dumpepisodes" value="false"/>
    <!-- 
      set updateAll to true if you want to refresh all information in your schedules with the infromation from theTvDb
      This is useful if a shows information is completely changed on theTvDb, or you suspect a bad run of the enricher.
    -->
    <add key="updateAll" value="false"/>
    <!--
      If you want to filter your recordings based on season number the only way possible for now is to use the description field with a contains filter.
      In your recordings you would set Description Contains "S02E" for example.
    -->
    <add key="episodeInDescription" value="false"/>
    <!-- 
      set updateTitle to true if you want to update any of your schedules programs with the subtitle that it is 
      matched with from thetvdb.  Note however that if something is incorrectly matched, that this will overwrite 
      the current title for one that may be incorrect...
    -->
    <add key="updateSubtitles" value="true"/>
  </appSettings>
  ```
#####Series Mapping
If you see errors for series that cannot be found, take a look at the *GuideEnricherService.exe.config* file in the program directory. You will be able to add series name mappings in there. Restart the service for changes to take effect.
```xml
<!--
      This section allows you to create mappings from names in the 
      guide data (i.e., SchedulesDirect) to names in thetvdb.com.

      You need this because the names are not always equal.  It comes
      in handy for shows that have the same name a really old shows....
      Like Chase on NBC needs to have a mapping to Chase (2010) to help 
      this tool find the proper series.

      Use the logs to help debug.

      If a regular name mapping does not work, you can use thetvdb.com
      show ID as a direct map as shown with $..! My Dad Says below.

      Also, you can use regular expressions to handle schedulesDirectNaming
      problems for series names that occasionally have too much information 
      in it.  The third example shows the use of a regex to tvdb.com name.
      The fourth example shows using a regex to tvdb.com id

      For shows that you record but know you're not going to find reliable
      data on thetvdb.com, you can just choose to ignore them.  See 
      the fifth entry for an example.  The tvdbComName still needs to be there
      for the XML parser, so just make it an empty string like the example.

    <seriesMap schedulesDirectName="$..! My Dad Says" tvdbComName="id=164951" />
    <seriesMap schedulesDirectName="regex=Boston Leg.*" tvdbComName="Boston Legal" />
    <seriesMap schedulesDirectName="regex=Boston Lega.*" tvdbComName="id=74058" />
    <seriesMap schedulesDirectName="regex=The Daily Show.*" tvdbComName="" ignore="true" />
   -->
  <seriesMapping>
    <seriesMap schedulesDirectName="Star Trek: Enterprise" tvdbComName="id=73893"/>
    <seriesMap schedulesDirectName="Flashpoint" tvdbComName="id=82438"/>
    <seriesMap schedulesDirectName="Dr. House" tvdbComName="id=73255"/>
    <seriesMap schedulesDirectName="Castle" tvdbComName="" ignore="true"/>
    <seriesMap schedulesDirectName="Monk" tvdbComName="" ignore="true"/>
  </seriesMapping>
```
#####Match Methods
You can change the priority of the match methods used for episode matching by changing the order. You can also comment out a method if you wish to disable it
```xml
  <MatchMethodsSection>
    <MatchMethods>
      <add name="GuideEnricher.EpisodeMatchMethods.AirDateMatchMethod"/>
      <!--      <add name="GuideEnricher.EpisodeMatchMethods.AbsoluteEpisodeNumberMatchMethod" />-->
      <add name="GuideEnricher.EpisodeMatchMethods.NumericSeasonEpisodeMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.EpisodeTitleMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.NoPunctuationMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.RemoveCommonWordsMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.InQuotesInDescriptionMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.FirstSentenceInDescriptionMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.DescriptionStartsWithEpisodeTitleMatchMethod"/>
      <add name="GuideEnricher.EpisodeMatchMethods.SeasonAndEpisodeInDescriptionMatchMethod"/>
    </MatchMethods>
  </MatchMethodsSection>
  ```
  
###Troubleshoot
Logs are written to the *guideenricher.log* file in the program directory. You can view them by simply double clicking the *guideenricher.log* file in your desired text editor.

Guide Enricher can also be started as a Console Application by double clicking *GuideEnricherService.exe* after you manually stopped the GuideEnricher Windows Service. The Console outputs all log events in real time.
