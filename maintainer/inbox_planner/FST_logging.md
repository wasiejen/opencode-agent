I used CONSTANTS.DEBUG<X> as ways to live debug via the console output.
But i think i used 4 different DEBUG values to be able to get infos from different parts of the functionality. but also added just anything that might have been useful.
So i would request research to on how to switch this to logging - not only blindly converting to logging, but with clear seperation. honestly it is a long time ago i implemented this - so there will be DEBUG output just for convenience or to solve a particular problem and stayed in the code after the fix.

CONSTANTS.DEBUG4: is the only real exception i can remember: is used for displaying the actual flow of pressed keys to replaced keys and to what keyactions it triggers via macros. In a formatted way to the hierachy when printed out is visiable -> indented and which versions od "<--" and "-->" to indicate control flow up and down the indentation.

i started to integrate logging. but i honestly do not even know if it would be useful and how exactly to use it to be more helpful than the DEBUG constants and direct output. output into different logging files could be useful maybe to track different functions and see problems in execution earlier? or how to instrument it to be used to catch error in the logic or to catch now silently dropped raised errors. so the research is honestly also about checking what would make sense and on how to use the logging to be helpful.

general goal ideas:
- how do instrument the whole repo to be helpful in finding causes of laggs, documenting errors and make it easier to accociate them to certain areas of code
  - how best to control the logging to now overwhelm me?
    - logg into different files dependent on a given start argument or loglevel via start argument?
