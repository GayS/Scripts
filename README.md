# Scripts
'MFIP Sanction/DWP Disqualification

'STATS GATHERING----------------------------------------------------------------------------------------------------
name_of_script = "NOTES - MFIP Sanction-DWP Disq.vbs"
start_time = timer

'FUNCTIONS----------------------------------------------------------------------------------------------------
'LOADING ROUTINE FUNCTIONS FROM GITHUB REPOSITORY---------------------------------------------------------------------------
url = "https://raw.githubusercontent.com/MN-Script-Team/BZS-FuncLib/master/MASTER%20FUNCTIONS%20LIBRARY.vbs"
SET req = CreateObject("Msxml2.XMLHttp.6.0")				'Creates an object to get a URL
req.open "GET", url, FALSE									'Attempts to open the URL
req.send													'Sends request
IF req.Status = 200 THEN									'200 means great success
	Set fso = CreateObject("Scripting.FileSystemObject")	'Creates an FSO
	Execute req.responseText								'Executes the script code
ELSE														'Error message, tells user to try to reach github.com, otherwise instructs to contact Veronica with details (and stops script).
	MsgBox 	"Something has gone wrong. The code stored on GitHub was not able to be reached." & vbCr &_ 
			vbCr & _
			"Before contacting Veronica Cary, please check to make sure you can load the main page at www.GitHub.com." & vbCr &_
			vbCr & _
			"If you can reach GitHub.com, but this script still does not work, ask an alpha user to contact Veronica Cary and provide the following information:" & vbCr &_
			vbTab & "- The name of the script you are running." & vbCr &_
			vbTab & "- Whether or not the script is ""erroring out"" for any other users." & vbCr &_
			vbTab & "- The name and email for an employee from your IT department," & vbCr & _
			vbTab & vbTab & "responsible for network issues." & vbCr &_
			vbTab & "- The URL indicated below (a screenshot should suffice)." & vbCr &_
			vbCr & _
			"Veronica will work with your IT department to try and solve this issue, if needed." & vbCr &_ 
			vbCr &_
			"URL: " & url
			script_end_procedure("Script ended due to error connecting to GitHub.")
END IF

'DIALOGS-------------------------------
'MFIP Sanction/DWP Disqualification Dialog Box
BeginDialog MFIP_Sanction_DWP_Disq_Dialog, 0, 0, 286, 305, "MFIP Sanction - DWP Disqualification"
  Text 5, 10, 70, 10, "MAXIS Case Number"
  EditBox 75, 5, 90, 15, case_number
  Text 5, 30, 80, 10, "HH Member's Number"
  EditBox 90, 25, 35, 15, HH_Member_Number
  Text 5, 50, 65, 10, "Type of Sanction:"
  CheckBox 70, 50, 85, 10, "Employment Services", ES_Sanction_Checkbox
  CheckBox 160, 50, 85, 10, "Child Support", Child_Support_Sanction_Checkbox
  Text 5, 70, 140, 10, "Effective Date of Sanction/Disqualification"
  EditBox 150, 65, 90, 15, Date_Sanction
  Text 5, 90, 75, 10, "Number of occurences"
  EditBox 90, 85, 30, 15, Number_Occurrences
  Text 5, 110, 60, 10, "Sanction Percent:"
  CheckBox 80, 110, 30, 10, "10%", ten_Percent_Checkbox
  CheckBox 120, 110, 30, 10, "30%", thirty_Percent_Checkbox
  CheckBox 160, 110, 30, 10, "100%", hundred_Percent_Checkbox
  Text 5, 135, 170, 10, "Last day to cure (10 day cutoff or last day of month)"
  EditBox 180, 130, 70, 15, Last_Day_Cure
  Text 5, 155, 105, 25, "Vendor information (if vendoring due to the sanction, vendor #, etc.) "
  EditBox 125, 155, 150, 25, Vendor_Information
  Text 5, 195, 95, 20, "Status update information was sent to:"
  CheckBox 105, 200, 90, 15, " Employment Services", Update_Sent_ES_Checkbox
  CheckBox 195, 200, 85, 15, "Child Care Assistance", Update_Sent_CCA_Checkbox
  CheckBox 5, 225, 30, 10, "FIAT", Fiat_Checkbox
  Text 5, 255, 70, 10, "Sign your case note"
  EditBox 80, 250, 90, 15, worker_signature
  ButtonGroup ButtonPressed
    OkButton 175, 275, 50, 15
    CancelButton 230, 275, 50, 15
EndDialog


'THE SCRIPT--------------------------------------------

'Connects to BlueZone
EMConnect ""

'Asks for Case Number
CALL MAXIS_case_number_finder(case_number)

'Shows dialog
DO
	DO
		DO
			Dialog MFIP_Sanction_DWP_Disq_Dialog
			IF ButtonPressed = 0 THEN StopScript
			IF worker_signature = "" THEN MsgBox "You must sign your case note!"
		LOOP UNTIL worker_signature <> ""
		IF IsNumeric(case_number) = FALSE THEN MsgBox "You must type a valid numeric case number!"
	LOOP UNTIL IsNumeric(case_number) = TRUE
	IF IsDate(Date_Sanction) = FALSE THEN MsgBox "You must type a valid date of sanction!"
LOOP UNTIL IsDate(Date_Sanction) = TRUE

'Checks MAXIS for password prompt
MAXIS_check_function

'Navigates to case note
CALL navigate_to_screen("CASE", "NOTE")

'Send PF9 to case note
PF9


'Writes case note
CALL write_new_line_in_case_note("***MFIP Sanction/DWP Disqualification***")
IF HH_Member_Number <> "" THEN CALL write_editbox_in_case_note("HH Member's Number", HH_Member_Number, 6)
IF ES_Sanction_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Employment Services sanction")
IF Child_Support_Sanction_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Child Support sanction")
IF Date_Sanction <> "" THEN CALL write_editbox_in_case_note("Effective date of sanction/disqualification", Date_Sanction, 6)
IF Number_Occurrences <> "" THEN CALL write_editbox_in_case_note("Number of occurences", Number_Occurrences, 6)
IF Ten_Percent_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Sanction Percent is 10%")
IF Thirty_Percent_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Sanction Percent is 30%")
IF Hundred_Percent_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Sanction Percent is 100%")
IF Last_Day_Cure <> "" THEN CALL write_editbox_in_case_note("Last day to cure", Last_Day_Cure, 6)
IF Vendor_Information <> "" THEN CALL write_editbox_in_case_note("Vendoring information", Vendor_Information, 6)
IF Update_Sent_ES_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Status update information was sent to Employment Services")
IF Update_Sent_CCA_Checkbox = 1 THEN CALL write_new_line_in_case_note("* Status update information was sent to Child Care Assistance")
IF FIAT_Checkbox = 1 THEN CALL write_new_line_in_case_note("* FIATed")



'case note worker signature
CALL write_new_line_in_case_note("---")
CALL write_new_line_in_case_note(worker_signature)




