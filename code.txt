
  I am using 2 demensional array to solve the problem.
  Here is the sketch of it
  scheduleArray = [
    [null,       1000 day1,    1030 day1,   1100 day1 ... 500 day1,   1000 day2,   1030 day2,   1100 day2 ... 500 day2]
    [Student 1,      -,            -,            -,            +,              +,            +,            +,            +    ]
    [Student 2,      +,            +,            -,            +,              -,            -,            +,            +    ]     
    [Student 3,      +,            -,            +,            +,              +,            +,            +,            +    ]
    [Student 4,      +,            +,            +,            +,              +,            +,            -,            +    ]
    [Student 5,      -,            -,            +,            -,              -,            +,            +,            -    ]
    ...
  ];

  The + sighn represent available time slot
  The - sighn represents unavailable time slot (the time slot is even blacklisted or taken)


Global constant variables
const sheet = SpreadsheetApp.getActiveSheet();
const numberOfStudents = 44;
const numberOfTimeSlotsTotal = 28;
const numberOfTimeSlotsDay = numberOfTimeSlotsTotal2;
const numberOfPods = 3;
const numberOfStudentsInPod = 15;
const numberOfCompanies = 20;
const numberOfCompaniesPerDay = 10;

Total students for each company.
const studentsToPick = 8;

The schedule represented by 2D array
var schedulingArray = Array.from(Array(numberOfStudents+1), () = new Array(numberOfTimeSlotsTotal+1));
schedulingArray[0][0] = !;

Array to hold time periods
const timePeriods = [1000AM Day 1, 1030AM Day 1, 1100AM Day 1, 1130AM Day 1, 1200PM Day 1, 1230PM Day 1, 100PM Day 1, 130PM Day 1, 200PM Day 1, 230PM Day 1, 300PM Day 1, 330PM Day 1, 400PM Day 1, 430PM Day 1,
                     1000AM Day 2, 1030AM Day 2, 1100AM Day 2, 1130AM Day 2, 1200PM Day 2, 1230PM Day 2, 100PM Day 2, 130PM Day 2, 200PM Day 2, 230PM Day 2, 300PM Day 2, 330PM Day 2, 400PM Day 2, 430PM Day 2
];

Two interview dates
var firstDay = new Date(482021);
firstDay = Utilities.formatDate(firstDay, GMT, EEEE, MMMM d, yyyy);

var secondDay = new Date(492021);
secondDay = Utilities.formatDate(secondDay, GMT, EEEE, MMMM d, yyyy);



  errorCell have white backgorund by default.
  If backgorund is red student have an interview with the same company more than 1 time OR company have more than one student at the given time slot.

  errorLogCell cell would contain an explanation of the issue.
  For example, if a student have interview with the same company twice, the error log cell will contain the name of that student,
  and the text saying that the student have interview with the same company twice.

  runErrorCheckCell cell contains the check box. The check box in uncheked by default.
  If you check the box manually, you will triger onEdit() function, which will call errorDataCheckOnEdit() function which will check for any conflictserrors on the sheet schedule

  runErrorCheckCellStatus display the status of errorDataCheckOnEdit() function. If the schedule is being checked at the moment, then runErrorCheckCellStatus 
  will display the text In progress.... When errorDataCheckOnEdit() finish running, the displayed text will be Done checking.
  If errorDataCheckOnEdit() was not called yet, then the runErrorCheckCellStatus text will be empty string.

  algoResultCheck cell would contain a list of students and the companies to which students can not be assigned due to the lack of available time slots.


var errorCell = sheet.getRange(M100);
var errorLogCell = sheet.getRange(N100);
var runErrorCheckCell = sheet.getRange(O100);
var runErrorCheckCellStatus = sheet.getRange(O101);
var algoResultCheck = sheet.getRange(Q100T100);

That variable holds the color which represent unavailable time slot
const unavailableColor = sheet.getRange('L100').getBackground();

--------------------------------- Sheet ranges -------------------------------------
Ranges of first names and last names
const firstNamesRangeDay1 = sheet.getRange(A3A46);
const lastNamesRangeDay1 = sheet.getRange(B3B46);

const firstNamesRangeDay2 = sheet.getRange(A51A94);

Array of companies and its range
const companiesRange = sheet.getRange(C100C119).getValues();
var companiesArray = [];
for (var companyCount = 0; companyCount  companiesRange.length; companyCount++)
{
  companiesArray.push(companiesRange[companyCount][0]);
}

Company Email Range
const companyEmailRange = sheet.getRange(R51R52);

Student Emails Range
const studentEmailsRange = sheet.getRange(R3R4);

If you want to fill PODs randomly, set it to True
const fillPods = true;

Range for the POD 1
const pod1Range = sheet.getRange(D100K106);
const pod1Range = sheet.getRange(D100I106);

Range for the POD 2
const pod2Range = sheet.getRange(D107K113);
const pod2Range = sheet.getRange(D107I113);

Range for the POD 3
const pod3Range = sheet.getRange(D114K119);
const pod3Range = sheet.getRange(D114I119);

Range for all PODs all together
const allPodsRange = sheet.getRange(D100K119);
const allPodsRange = sheet.getRange(D100I119);



  If you want to use more pods, you can add them above. Dont forget to change const values
  at the beginning of the code and call fillPodWithStudents() as many times as you have PODs

---------------------------------------------------------------------------------------

Here is body text for emails. The string with the schedule is separate.
const studentEmailText = Katherine Chuang, thank you for your time and patience, it was nice to have a Data Structures class with you, good luck!nn;

const companyEmailText = Allan James Lapid, thank you for your time and patience, it was nice to work with you, good luck!nn;

This function takes all of time slots on the sheet and makes two-dimensional array representation of that sheet
function fillSchedulingArray()
{
  Fill time periods into the schedule
  for (timeSlot = 1; timeSlot = numberOfTimeSlotsTotal; timeSlot++)
  {
    schedulingArray[0][timeSlot] =  timePeriods[timeSlot-1];
  }

  Initialize loop functiuon to be able to go through each student and his info
  for (studentCount = 1; studentCount = numberOfStudents; studentCount++)
  {
    Fill in student names
    schedulingArray[studentCount][0] = firstNamesRangeDay1.getCell(studentCount, 1).getValue() +   + lastNamesRangeDay1.getCell(studentCount, 1).getValue();

    Get current Student's row in range
    var studentRowRangeDay1 = sheet.getRange(firstNamesRangeDay1.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);
    var studentRowRangeDay2 = sheet.getRange(firstNamesRangeDay2.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);
    

     Since there is no possibility to merge 2 ranges in a way I need,
       I will have to go through each day individually.
       If a time slot is empty and available, mark it with + sign
       If a time slot is unavailable mark it with - sighn
       If time slot is alredy taken by a company, mark it with the company name text
     
    
    for (var day = 1; day = 2; day++)
    {
      for (var timeSlot = 1; timeSlot = numberOfTimeSlotsDay; timeSlot++)
      {
        if (day == 1)
        {
          if (studentRowRangeDay1.getCell(1, timeSlot+2).getBackground() == unavailableColor)
          {
            schedulingArray[studentCount][timeSlot] = -;
          }
          else if (studentRowRangeDay1.getCell(1, timeSlot+2).getValue() != )
          {
            schedulingArray[studentCount][timeSlot] = studentRowRangeDay1.getCell(1, timeSlot+2).getValue();
          }
          else if (studentRowRangeDay1.getCell(1, timeSlot+2).getValue() == )
          {
            schedulingArray[studentCount][timeSlot] = +;
          }
          else
          {
            Logger.log(INVALID CELL VALUE)
          }
        }

        else if (day == 2)
        {
          if (studentRowRangeDay2.getCell(1, timeSlot+2).getBackground() == unavailableColor)
          {
            schedulingArray[studentCount][timeSlot+numberOfTimeSlotsDay] = -;
          }
          else if (studentRowRangeDay2.getCell(1, timeSlot+2).getValue() != )
          {
            schedulingArray[studentCount][timeSlot+numberOfTimeSlotsDay] = studentRowRangeDay2.getCell(1, timeSlot+2).getValue();
          }
          else if (studentRowRangeDay2.getCell(1, timeSlot+2).getValue() == )
          {
            schedulingArray[studentCount][timeSlot+numberOfTimeSlotsDay] = +;
          }
          else
          {
            Logger.log(INVALID CELL VALUE)
          }
        }
      }
    }
  }
}


  First, this function fill PODs with students randomly (can be turned OFF by setting 'fillPods' variable to false) on the sheet and on the 2 dimensional array representation.
  After that, this function calls findFillOpenTimeSlot(student, company) for each student. That function look for available non-conflict time slot on the 2 dimensional array representation.
  Then, schedulingAlgorithm() copy all of the data from 2 dimensional array representation to the actiuall sheet.
  Before the function ends, it calls errorDataCheckOnEdit() to be sure there is no conflicts on the schedule. Its not really needed since the algorithm is perfect, but I feel more confident that way. 

function schedulingAlgorithm()
{
  fillSchedulingArray();
  
  Generate pods
  numberStudentsToAdd = numberOfStudentsInPod;
  remainingStudents = numberOfStudents;
  podsArray = [];

  for (var podCount = 0; podCount  numberOfPods; podCount++)
  {
    podsArray.push([]);
    if (remainingStudents  numberOfStudentsInPod)
    {
      numberStudentsToAdd = remainingStudents;
    }

    for (studentCount = 0; studentCount  numberStudentsToAdd; studentCount++)
    {
      podsArray[podCount][studentCount] = schedulingArray[((studentCount+1)+(numberOfStudentsInPod  podCount))][0];
    }
    calculateRemaningStudents();
  }


  
  Fill Pods on the SHEET
  if (fillPods == true)
  {
    (pod number, number of companies in a pod, number of students to pick, pod range on the sheet)
    fillPodWithStudents(0, 7, studentsToPick, pod1Range);
    
    fillPodWithStudents(1, 7, studentsToPick, pod2Range);
    
    fillPodWithStudents(2, 6, studentsToPick, pod3Range);

    Add more PODs if you need
  }

  
  Fill PODs in the array
  var allPodsValues = allPodsRange.getValues();

  Logger.log(allPodsValues);
  Logger.log(companiesArray);

  Fill time slots on the scheduling array
  for (var companyCount = 0; companyCount  numberOfCompanies; companyCount++)
  {
    for (var studentCount = 0; studentCount  studentsToPick; studentCount++)
    {
      findFillOpenTimeSlot(allPodsValues[companyCount][studentCount], companiesArray[companyCount]);
    }
  }

  Fill students in the table using shceduling array
  for (var studentCount = 1; studentCount = numberOfStudents; studentCount++)
  {
    var studentRowRangeDay1 = sheet.getRange(firstNamesRangeDay1.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);
    var studentRowRangeDay2 = sheet.getRange(firstNamesRangeDay2.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);

    for (var day = 1; day = 2; day++)
    {
      for (var timeSlot = 1; timeSlot = numberOfTimeSlotsDay; timeSlot++)
      {
        if (day == 1)
        {
          if (schedulingArray[studentCount][timeSlot] != + && schedulingArray[studentCount][timeSlot] != -)
          {
            studentRowRangeDay1.getCell(1, timeSlot+2).setValue(schedulingArray[studentCount][timeSlot]);
          }
        }

        else if (day == 2)
        {
          if (schedulingArray[studentCount][timeSlot + numberOfTimeSlotsDay] != + && schedulingArray[studentCount][timeSlot + numberOfTimeSlotsDay] != -)
          {
            studentRowRangeDay2.getCell(1, timeSlot+2).setValue(schedulingArray[studentCount][timeSlot + numberOfTimeSlotsDay]);
          }
        }
      }
    }
  }

  
  Logger.log(schedulingArray);
  
  Checks
  errorCell.setBackground(white);
  errorLogCell.setValue();
  runErrorCheckCellStatus.setValue();
  
  runErrorCheckCellStatus.setValue(In progress...);
  errorDataCheckOnEdit();
  runErrorCheckCellStatus.setValue(Done checking);
  
}

Calls every time when sheet is being changed MANUALLY. However, the error check process only initializing when 'runErrorCheck' check-box is checked by the user.
function onEdit(e)
{
  errorCell.setBackground(white);
  runErrorCheckCellStatus.setValue();
  errorLogCell.setValue();
  if (runErrorCheckCell.isChecked() == true)
  {
    runErrorCheckCellStatus.setValue(In progress...);
    errorDataCheckOnEdit();
    runErrorCheckCellStatus.setValue(Done checking);
    runErrorCheckCell.uncheck();
  }
}


  Actually very imporatnt function. In my example there is 44 students and 3 pods with 15 students.
  This function makes it possible to have 2 pods with 15 student and one pod with 14 students. So, it does not matter how many studentspoods you have. The algorithm would work perfectly with any numbers.

function calculateRemaningStudents()
{
  remainingStudents = remainingStudents - numberStudentsToAdd;
}

Fill pods on the SHEET
function fillPodWithStudents(podNumber, companiesInPod, studentsInPod, sheetRange)
{
  var podRange = sheetRange;
  var podRangeValues = podRange.getValues();
  for (var companyCount = 0; companyCount  companiesInPod; companyCount++)
  {
    for (var studentCount = 0; studentCount  studentsInPod; studentCount++)
    {
      do
      {
        var randomInt = Math.floor(Math.random()  14);
        if (podRangeValues[companyCount].indexOf(podsArray[podNumber][randomInt]) != -1)
        {
          duplicate = true;
        }
        else
        {
          podRangeValues[companyCount][studentCount] = podsArray[podNumber][randomInt];
          duplicate = false;
        }
      }
      while (duplicate == true);
    }
  }
  podRange.setValues(podRangeValues);
}

Return true if passed array have duplicates in it. Return false if no duplicates were found.
function findDuplicates(array)
{
  var checkedValuesArr = [];
    for (var i = 0; i  array.length; ++i) 
    {
      value = array[i];
      if (checkedValuesArr.indexOf(value) != -1)
      {
        return true;
      }
      if (value != + && value != -)
      {
        checkedValuesArr.push(value);
      }
    }
    return false;
}


  Thins function is only called in findFillOpenTimeSlot() function. Don't call it in main algorithm.
  This function calls everytime when shceduling algorithm tries to fill the time slot.
  This function will not let algorithm fill the time slot if that would create a conflicterror in the schedule.

function errorDataCheck()
{
  var error = false;
  Check if a student have an interview with the same company more than 1 time in 2 days.
  for (var studentCount = 1; studentCount = numberOfStudents; studentCount++)
  {
    if (findDuplicates(schedulingArray[studentCount]) == true)
    {
      error = true;
    }
  }

  Check if company have to interview more than one student at a given time slot at a given date.
  for (var timePeriodCount = 1; timePeriodCount = numberOfTimeSlotsTotal; timePeriodCount++)
  {
    var timeSlotsForPeriodArr = [];
    
    for (var timeSlotCount = 1; timeSlotCount = numberOfStudents; timeSlotCount++)
    {
      timeSlotsForPeriodArr.push(schedulingArray[timeSlotCount][timePeriodCount]);
    }
    
    if (findDuplicates(timeSlotsForPeriodArr) == true)
    {
      error = true;
    }
  }

  return error;
}

Checks for every possbile conflict that you cant even imagine.
function errorDataCheckOnEdit()
{
  fillSchedulingArray();
  Check if a student have an interview with the same company more than 1 time in 2 days.
  for (var studentCount = 1; studentCount = numberOfStudents; studentCount++)
  {
    if (findDuplicates(schedulingArray[studentCount]) == true)
    {
      errorCell.setBackground(red);
      errorLogCell.setValue(errorLogCell.getValue() +   + (schedulingArray[studentCount][0] +  has an interview with the same company more than 1 time in 2 days. ));
    }
  }

  Check if company have to interview more than one student at a given time slot at a given date.
  for (var timePeriodCount = 1; timePeriodCount = numberOfTimeSlotsTotal; timePeriodCount++)
  {
    var timeSlotsForPeriodArr = [];
    
    for (var timeSlotCount = 1; timeSlotCount = numberOfStudents; timeSlotCount++)
    {
      timeSlotsForPeriodArr.push(schedulingArray[timeSlotCount][timePeriodCount])
    }
    
    if (findDuplicates(timeSlotsForPeriodArr) == true)
    {
      errorCell.setBackground(red);
      errorLogCell.setValue(errorLogCell.getValue() +   + (In time period  + schedulingArray[0][timePeriodCount] + the company has more than 1 interview. ));
    }
  }

  
  Check if a black listed time slot is assigned to any students
  Check if a time slot was assigned to the company that do not interview on that day.

  To be able to check that, the program will loop through all time slots.
  
  for (var studentCount = 1; studentCount = numberOfStudents; studentCount++)
  {
    var studentRowRangeDay1 = sheet.getRange(firstNamesRangeDay1.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);
    var studentRowRangeDay2 = sheet.getRange(firstNamesRangeDay2.getCell(studentCount, 1).getRowIndex(), 1, 1, numberOfTimeSlotsDay+2);
    for (var day = 1; day = 2; day++)
    {
      for (var timeSlot = 1; timeSlot = numberOfTimeSlotsDay; timeSlot++)
      {
        if (day == 1)
        {
          Check if a black listed time slot is assigned to any students on Day 1
          if (studentRowRangeDay1.getCell(1, timeSlot+2).getBackground() == unavailableColor && studentRowRangeDay1.getCell(1, timeSlot+2).getValue() != )
          {
            errorCell.setBackground(red);
            errorLogCell.setValue(errorLogCell.getValue() +   + (In time period  + schedulingArray[0][timeSlot] +  the black listed spot is assigned to the student. ));
          }
          
          Check if a time slot on Day 1 was asssigned to the company that do not interview on Day 1.
          if((studentRowRangeDay1.getCell(1, timeSlot+2).getValue() != ) && (companiesArray.lastIndexOf(studentRowRangeDay1.getCell(1, timeSlot+2).getValue()) + 1) % 2 == 0)
          {
            errorCell.setBackground(red);
            errorLogCell.setValue(errorLogCell.getValue() +   + (In time period  + schedulingArray[0][timeSlot] +  company  + studentRowRangeDay1.getCell(1, timeSlot+2).getValue() +  don't interview on that day. ));
          }
        }

        else if (day == 2)
        {
          Check if a black listed time slot is assigned to any students on Day 2
          if (studentRowRangeDay2.getCell(1, timeSlot+2).getBackground() == unavailableColor && studentRowRangeDay2.getCell(1, timeSlot+2).getValue() != )
          {
            errorCell.setBackground(red);
            errorLogCell.setValue(errorLogCell.getValue() +   + (In time period  + schedulingArray[0][timeSlot + numberOfTimeSlotsDay] +  the black listed spot is assigned to the student. ));
          }

          Check if a time slot on Day 2 was asssigned to the company that do not interview on Day 2.
          if((studentRowRangeDay2.getCell(1, timeSlot+2).getValue() != ) && (companiesArray.lastIndexOf(studentRowRangeDay2.getCell(1, timeSlot+2).getValue()) + 1) % 2 != 0)
          {
            errorCell.setBackground(red);
            errorLogCell.setValue(errorLogCell.getValue() +   + (In time period  + schedulingArray[0][timeSlot + numberOfTimeSlotsDay] +  company  + studentRowRangeDay2.getCell(1, timeSlot+2).getValue() +  don't interview on that day. ));
          }
        }
      }
    }
  }
}

Function fills open time slots on the shcedule Array, not the sheet
function findFillOpenTimeSlot(studentName, companyName)
{
  First find student index in the schedule 2D array by the student's name.
  var studentID = [];
  for (var itemCount = 0; itemCount  schedulingArray.length; itemCount++)
  {
    index = schedulingArray[itemCount].indexOf(studentName);
    if (index  -1)
    {
      studentID = [itemCount, index];
    }
  }

  var companyAssigned = false;
  
  Based on the company, decide to what day the interview should be assigned
  var day = 1;
  if ((companiesArray.indexOf(companyName) + 1) % 2 == 0 )
  {
    day = 2;
  }
  
  if (day == 1)
  {
    for (var timeSlotCount = 1; timeSlotCount = numberOfTimeSlotsDay; timeSlotCount++)
    {
      if (schedulingArray[studentID[0]][timeSlotCount] == +)
      {
        schedulingArray[studentID[0]][timeSlotCount] = companyName;
        if (errorDataCheck() == true)
        {
          schedulingArray[studentID[0]][timeSlotCount] = +;
        }
        else 
        {
          companyAssigned = true;
          break;
        }
      }
    } 
  }
  else if (day == 2)
  {
    for (var timeSlotCount = numberOfTimeSlotsDay + 1; timeSlotCount = numberOfTimeSlotsDay  2; timeSlotCount++)
    {
      if (schedulingArray[studentID[0]][timeSlotCount] == +)
      {
        schedulingArray[studentID[0]][timeSlotCount] = companyName;
        if (errorDataCheck() == true)
        {
          schedulingArray[studentID[0]][timeSlotCount] = +;
        }
        else 
        {
          companyAssigned = true;
          break;
        }
      }
    } 
  }

  If all of time slots were checked but interview was not assigned, that means were no available time slot for that student.
  if (companyAssigned == false)
  {
    Logger.log(I could not assign interview for  + studentName +  with  + companyName + . );
    algoResultCheck.setValue(algoResultCheck.getValue() +  I could not assign interview for  + studentName +  with  + companyName + . )
  }
}

Send email to all students. You want to double check these variables 'studentEmailsRange', 'companiesRange', 'studentEmailText'
Check Execution log, to see the schedules that were sent.
function sendEmailToStudents()
{
  fillSchedulingArray();
  const studentEmailsValues = studentEmailsRange.getValues();
  var studentSchedule = ;

  for (var studentCount = 1; studentCount = studentEmailsValues.length; studentCount++)
  {
    for (var timeSlots = 1; timeSlots = numberOfTimeSlotsTotal; timeSlots++)
    {
      if (schedulingArray[studentCount][timeSlots].length  1)
      {
        if (schedulingArray[0][timeSlots].includes(Day 1) == true)
        {
          studentSchedule += firstDay +  from  + (schedulingArray[0][timeSlots].substring(0, schedulingArray[0][timeSlots].length - 6)) +  EST to  + (schedulingArray[0][timeSlots+1].substring(0, schedulingArray[0][timeSlots+1].length - 6)) +  EST ;
          studentSchedule += schedulingArray[studentCount][timeSlots] + nn;
        }
        else if (schedulingArray[0][timeSlots].includes(Day 2) == true)
        {
          studentSchedule += secondDay +  from  + (schedulingArray[0][timeSlots].substring(0, schedulingArray[0][timeSlots].length - 6)) +  EST to  + (schedulingArray[0][timeSlots+1].substring(0, schedulingArray[0][timeSlots+1].length - 6)) +  EST ;
          studentSchedule += schedulingArray[studentCount][timeSlots] + nn;
        }
      }  
    }
    Logger.log(studentEmailsValues[studentCount-1][0]);
    Logger.log(studentSchedule);
    
    (email, subject, body)
    var email = studentEmailsValues[studentCount-1][0];
    var subject = Response Needed TTP Residency;
    var message = studentEmailText + studentSchedule;
    MailApp.sendEmail(email, subject, message, {
      name Allan James Lapid,
      attachments [attachment.next().getAs(MimeType.PDF)],
      replyTo replyToEmail
    });

    studentSchedule = ;
  }
}

Send email to all companies. You want to double check these variables 'companyEmailsRange', 'companiesRange', 'companyEmailText'
Check Execution log, to see the schedules that were sent.
function sendEmailToCompanies()
{
  fillSchedulingArray();
  const companyEmailsValues = companyEmailRange.getValues();
  var companySchedule = ;

  for (var companyCount = 0; companyCount  companyEmailsValues.length; companyCount++)
  {
    for (var studentCount = 1; studentCount = numberOfStudents; studentCount++)
    {
      for (var timeSlots = 1; timeSlots = numberOfTimeSlotsTotal; timeSlots++)
      {
        if (schedulingArray[studentCount][timeSlots] == companiesArray[companyCount])
        {
          if (schedulingArray[0][timeSlots].includes(Day 1) == true)
          {
            companySchedule += firstDay +  from  + (schedulingArray[0][timeSlots].substring(0, schedulingArray[0][timeSlots].length - 6)) +  EST to  + (schedulingArray[0][timeSlots+1].substring(0, schedulingArray[0][timeSlots+1].length - 6)) +  EST ;
            companySchedule += schedulingArray[studentCount][0] + nn;
          }
          else if (schedulingArray[0][timeSlots].includes(Day 2) == true)
          {
            companySchedule += secondDay +  from  + (schedulingArray[0][timeSlots].substring(0, schedulingArray[0][timeSlots].length - 6)) +  EST to  + (schedulingArray[0][timeSlots+1].substring(0, schedulingArray[0][timeSlots+1].length - 6)) +  EST ;
            companySchedule += schedulingArray[studentCount][0] + nn;
          }
        }  
      }
    }
    Logger.log(companiesArray[companyCount]);
    Logger.log(companySchedule);

    (email, subject, body)
    var email = companyEmailsValues[companyCount][0];
    var subject = Response Needed TTP Residency;
    var message = companyEmailText + companySchedule;
    var attachment = DriveApp.getFilesByName('Quiz 1 MATH 1026(1)_16977.pdf')
    MailApp.sendEmail(email, subject, message, {
      name Allan James Lapid,
      attachments [attachment.next().getAs(MimeType.PDF)],
      replyTo replyToEmail
    });    

    companySchedule = ;
  }
}