#Python-CERR
#Python for CERR processing. ArcMap Data Driven Pages

===========

### Python script for producing maps as supporting documentation for the CERR Challenge Process
### Version 2.00 06/04/2014 (Modified Post 2014 Challenge Process)


# Import Modules 
import arcpy, os, sys

# Change environment settings to allow output overwrite
arcpy.env.overwriteOutputs = True

# Set Directory Paths (how to make as script parameters??)
outDir = r'S:\IT\GIS\Liz\Python\CERR\Maps'
mxdpath = r'S:\IT\GIS\Liz\Python\CERR\Maps\PythonCERR.mxd'

# Set Output PDF
pdfPath = outDir + r"\FinalMapBook.pdf"
if os.path.exists(pdfPath): # Check to see if file already exists, delete if it does
    os.remove(pdfPath)
finalPdf = arcpy.mapping.PDFDocumentCreate(pdfPath) # creates obj in memory. path string specifies name when saveAndClose method is called

# Set Script Parameters
inputLayer = arcpy.GetParameterAsText(0) #Select Data Driven Page layer used for map creation
##outputLocation = arcpy.GetParameterAsText(1) ??? HOW DO I SET THIS PART UP???

# Set up Map Document Variables
mxd = arcpy.mapping.MapDocument(mxdpath)
ddp = mxd.dataDrivenPages
dataframes = arcpy.mapping.ListDataFrames(mxd, '')
actDataframe = arcpy.mapping.ListDataFrames(mxd, '')[0]
#inputLayer = r'S:\IT\GIS\Projects\Serverance\working\2014\CERR_2014_Scratch.gdb\SpatJoin_05012014'

inputTable = r'S:\IT\GIS\Liz\Python\CERR\Data\Python_CERR.gdb\MC_TAC_AgencyCodes'
tempTable = r'S:\IT\GIS\Liz\Python\CERR\Data\Python_CERR.gdb\tempTable'

# Set field variables
namefield = 'Match_addr'
tacfield = 'TAC'
qMuni_County = 'Muni_County'
qDistrict = 'District'
filename = 'filename'

# Set layout elements
lblGrandJunction = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblGrandJunction")[0]
lblFruita = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblFruita")[0]
lblCollbran = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblCollbran")[0]
lblDebeque = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblDebeque")[0]
lblPalisade = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblPalisade")[0]
lblCounty = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "lblCounty")[0]

# Set layout elements
txtGrandJunction = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtGrandJunction")[0]
txtFruita = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtFruita")[0]
txtCollbran = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtCollbran")[0]
txtDebeque = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtDebeque")[0]
txtPalisade = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtPalisade")[0]
txtCounty = arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "txtCounty")[0]

# Set scale bar variables
scale1 = arcpy.mapping.ListLayoutElements(mxd, "MAPSURROUND_ELEMENT", "scale1")[0]
scale2 = arcpy.mapping.ListLayoutElements(mxd, "MAPSURROUND_ELEMENT", "scale2")[0]
scale3 = arcpy.mapping.ListLayoutElements(mxd, "MAPSURROUND_ELEMENT", "scale3")[0]
scale4 = arcpy.mapping.ListLayoutElements(mxd, "MAPSURROUND_ELEMENT", "scale4")[0]

# Reference dynamic table elements 
for elm in arcpy.mapping.ListLayoutElements(mxd):
    if elm.name == "horzline": horzline = elm
    if elm.name == "vertline": vertline = elm
    if elm.name == "celltxt":  cellTxt = elm
    if elm.name == "headertxt": headerTxt = elm
    if elm.name == "txt_MatchAddr": txt_MatchAddr = elm

#Clean-up before next page (deletes cellTxt values & extra lines)
##arcpy.Delete_management("in_memory\select1")
##arcpy.Delete_management("in_memory\sort1")
##arcpy.DeleteRows_management(tempTable)

# Loop through Data Driven Pages
for pgNum in range(1, ddp.pageCount + 1):
    ddp.currentPageID = pgNum # set the current page
    pgName = ddp.pageRow.getValue(namefield) # get the page name
    pgTAC = ddp.pageRow.getValue(tacfield) # get the TAC value (for TAC table)
    pgLabel = ddp.pageRow.getValue(filename) # get the page label (for output pdf)

    if txt_MatchAddr.elementWidth > 2.5:
        txt_MatchAddr.fontSize = 11.8
        txt_MatchAddr.elementPositionX = 4.26
        txt_MatchAddr.elementPositionY = 2.27
    else:
        txt_MatchAddr.fontSize = 14
        txt_MatchAddr.elementPositionX = 4.26
        txt_MatchAddr.elementPositionY = 2.27        

    # Build selection set
    cursor = arcpy.da.SearchCursor(inputTable, ('TAC_Code', 'Year_', 'Agency_Name'), ''' "Year_" = 2013 ''')
    insertcursor = arcpy.da.InsertCursor(tempTable, ('TAC_Code', 'Year_', 'Agency_Name'))
 
    for row in cursor:
        if row[0] == pgTAC:
            insertcursor.insertRow(row)                     
            del row
            del insertcursor
            del cursor
            cursor = arcpy.da.SearchCursor(inputTable, ('TAC_Code', 'Year_', 'Agency_Name'), ''' "Year_" = 2013 ''')
            insertcursor = arcpy.da.InsertCursor(tempTable, ('TAC_Code', 'Year_', 'Agency_Name'))

    arcpy.TableSelect_analysis(tempTable, "in_memory\select1")
    numRecords = int(arcpy.GetCount_management("in_memory\select1").getOutput(0))
    arcpy.Sort_management("in_memory\select1", "in_memory\sort1", [["Agency_Name", "ASCENDING"]])

    #Graphic table variable values
    tableHeight = 2.45
    tableWidth = 2.68
    headerHeight = 0.2
    rowHeight = 0.15
    upperX = 0.25
    upperY = 2.65

    #Add note if no records
    if numRecords == 0:
        print "nadda"
    else:
        #if number of rows exceeds page space, resize row height
        if ((tableHeight - headerHeight) / numRecords) < rowHeight:
            headerHeight = headerHeight * ((tableHeight - headerHeight) / numRecords) / rowHeight
            rowHeight = (tableHeight - headerHeight) / numRecords
      
    #Set and clone vertical line work
    vertline.elementHeight = (headerHeight) + (rowHeight * (numRecords -1))
    vertline.elementPositionX = upperX
    vertline.elementPositionY = upperY

    temp_vert = vertline.clone("_clone")
    temp_vert.elementPositionX = upperX - 1.5
    temp_vert = vertline.clone("_clone")
    temp_vert.elementPositionY = upperY - 2.4
     
    #Set and clone horizontal line work
    horzline.elementWidth = tableWidth
    horzline.elementPositionX = upperX
    horzline.elementPositionY = upperY

    horzline.clone("_clone")
    horzline.elementPositionY = upperY - headerHeight

    print numRecords
    y=upperY - headerHeight
    for horz in range(1, numRecords+1):
        y = y - rowHeight
        temp_horz = horzline.clone("_clone")
        temp_horz.elementPositionY = y

    #Set header text elements
    headerTxt.fontSize = headerHeight / 0.0155
    headerTxt.text = "Taxing Authority Detail"
    headerTxt.elementPositionX = upperX
    headerTxt.elementPositionY = upperY + (headerHeight / 2)

    newFieldTxt = headerTxt.clone("_clone")
    newFieldTxt.text = "TAC Code"
    newFieldTxt.elementPositionX = upperX + 2.1

    #Set and clone cell text elements
    cellTxt.fontSize = rowHeight / 0.0155

    y = upperY - headerHeight
    rows = arcpy.SearchCursor("in_memory\sort1")
    for row in rows:
        x = upperX + 0.05
        col1CellTxt = cellTxt.clone("_clone")
        col1CellTxt.text = row.getValue("Agency_Name")
        col1CellTxt.elementPositionX = x
        col1CellTxt.elementPositionY = y
        col2CellTxt = cellTxt.clone("_clone")
        col2CellTxt.text = row.getValue("TAC_Code")
        col2CellTxt.elementPositionX = x + 2.3
        col2CellTxt.elementPositionY = y
        y = y - rowHeight
        if col1CellTxt.elementWidth > 2:
            col1CellTxt.elementWidth = 2
            

    # Set scale bar based on page extent
    if actDataframe.scale < 4000:
        scale1.elementPositionX = 0.33
        scale1.elementPositionY = 2.98
        scale2.elementPositionX = 11.6
        scale3.elementPositionX = 11.6
        scale4.elementPositionX = 11.6
    elif 4001 <= actDataframe.scale <= 10000:
        scale1.elementPositionX = 11.6
        scale2.elementPositionX = 0.33
        scale2.elementPositionY = 2.98
        scale3.elementPositionX = 11.6
        scale4.elementPositionX = 11.6
    elif 10001 <= actDataframe.scale <= 20000:
        scale1.elementPositionX = 11.6
        scale2.elementPositionX = 11.6
        scale3.elementPositionX = 0.33
        scale3.elementPositionY = 2.98
        scale4.elementPositionX = 11.6    
    elif actDataframe.scale > 20001:
        scale1.elementPositionX = 11.6
        scale2.elementPositionX = 11.6
        scale3.elementPositionX = 11.6
        scale4.elementPositionX = 0.33
        scale4.elementPositionY = 2.98

    # Set variables to set data frame insets
    reportedDistrict = ddp.pageRow.getValue(qMuni_County)
    newDistrict = ddp.pageRow.getValue(qDistrict)
     
    if reportedDistrict == 'City of Grand Junction':
        
        lblGrandJunction.elementPositionX = 8.27
        lblGrandJunction.elementPositionY = 2.63
        lblFruita.elementPositionX = 10.0
        lblCollbran.elementPositionX = 10.0
        lblDebeque.elementPositionX = 10.0
        lblPalisade.elementPositionX = 10.0
        lblCounty.elementPositionX = 12.0

        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0

        for frames in dataframes:
            if frames.name == 'df_GJT':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    elif reportedDistrict == 'City of Fruita':

        lblGrandJunction.elementPositionX = 10.0
        lblFruita.elementPositionY = 2.63
        lblFruita.elementPositionX = 8.27
        lblCollbran.elementPositionX = 10.0
        lblDebeque.elementPositionX = 10.0
        lblPalisade.elementPositionX = 10.0
        lblCounty.elementPositionX = 12.0

        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0
        
        for frames in dataframes:
            if frames.name == 'df_Fruita':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    elif reportedDistrict == 'Town of Collbran':

        lblGrandJunction.elementPositionX = 10.0
        lblFruita.elementPositionX = 10.0
        lblCollbran.elementPositionX = 8.27
        lblCollbran.elementPositionY = 2.63
        lblDebeque.elementPositionX = 10.0
        lblPalisade.elementPositionX = 10.0
        lblCounty.elementPositionX = 12.0

        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0
        
        for frames in dataframes:
            if frames.name == 'df_Collbran':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    elif reportedDistrict == 'Town of De Beque':

        lblGrandJunction.elementPositionX = 10.0
        lblFruita.elementPositionX = 10.0
        lblCollbran.elementPositionX = 10.0
        lblDebeque.elementPositionX = 8.27
        lblDebeque.elementPositionY = 2.63
        lblPalisade.elementPositionX = 10.0
        lblCounty.elementPositionX = 12.0

        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0
        
        for frames in dataframes:
            if frames.name == 'df_Debeque':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    elif reportedDistrict == 'Town of Palisade':

        lblGrandJunction.elementPositionX = 10.0
        lblFruita.elementPositionX = 10.0
        lblCollbran.elementPositionX = 10.0
        lblDebeque.elementPositionX = 10.0
        lblPalisade.elementPositionX = 8.27
        lblPalisade.elementPositionY = 2.63
        lblCounty.elementPositionX = 12.0
        
        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0
        
        for frames in dataframes:
            if frames.name == 'df_Palisade':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    elif reportedDistrict =='Mesa County':

        if newDistrict == 'City of Fruita':
            lblGrandJunction.elementPositionX = 10.0
            lblFruita.elementPositionY = 2.63
            lblFruita.elementPositionX = 8.27
            lblCollbran.elementPositionX = 10.0
            lblDebeque.elementPositionX = 10.0
            lblPalisade.elementPositionX = 10.0
            lblCounty.elementPositionX = 12.0

            txtCounty.elementPositionX = 10.0
            txtGrandJunction.elementPositionX = 10.0
            txtFruita.elementPositionX = 3.18
            txtFruita.elementPositionY = 1.07
            txtCollbran.elementPositionX = 10.0
            txtDebeque.elementPositionX = 10.0
            txtPalisade.elementPositionX = 10.0
        
            for frames in dataframes:
                if frames.name == 'df_Fruita':
                    frames.elementPositionX = 5.7
                    frames.elementPositionY = 0.5
                elif frames.name == 'df_Address':
                    frames.elementPositionX = 0.2745
                    frames.elementPositionY = 2.9452
                else:
                    frames.elementPositionX = 8.8
                    frames.elementPositionY = 0.7

        elif newDistrict == 'City of Grand Junction':
            lblGrandJunction.elementPositionX = 8.27
            lblGrandJunction.elementPositionY = 2.63
            lblFruita.elementPositionX = 10.0
            lblCollbran.elementPositionX = 10.0
            lblDebeque.elementPositionX = 10.0
            lblPalisade.elementPositionX = 10.0
            lblCounty.elementPositionX = 12.0

            txtCounty.elementPositionX = 10.0
            txtGrandJunction.elementPositionX = 3.18
            txtGrandJunction.elementPositionY = 1.07
            txtFruita.elementPositionX = 10.0
            txtCollbran.elementPositionX = 10.0
            txtDebeque.elementPositionX = 10.0
            txtPalisade.elementPositionX = 10.0

            for frames in dataframes:
                if frames.name == 'df_GJT':
                    frames.elementPositionX = 5.7
                    frames.elementPositionY = 0.5
                elif frames.name == 'df_Address':
                    frames.elementPositionX = 0.2745
                    frames.elementPositionY = 2.9452
                else:
                    frames.elementPositionX = 8.8
                    frames.elementPositionY = 0.7
                    
        elif newDistrict == 'Town of Palisade':
           
            lblGrandJunction.elementPositionX = 10.0
            lblFruita.elementPositionX = 10.0
            lblCollbran.elementPositionX = 10.0
            lblDebeque.elementPositionX = 10.0
            lblPalisade.elementPositionX = 8.27
            lblPalisade.elementPositionY = 2.63
            lblCounty.elementPositionX = 12.0

            txtCounty.elementPositionX = 10.0
            txtGrandJunction.elementPositionX = 10.0
            txtFruita.elementPositionX = 10.0
            txtCollbran.elementPositionX = 10.0
            txtDebeque.elementPositionX = 10.0
            txtPalisade.elementPositionX = 3.18
            txtPalisade.elementPositionY = 1.07
            
            for frames in dataframes:
                if frames.name == 'df_Palisade':
                    frames.elementPositionX = 5.7
                    frames.elementPositionY = 0.5
                elif frames.name == 'df_Address':
                    frames.elementPositionX = 0.2745
                    frames.elementPositionY = 2.9452
                else:
                    frames.elementPositionX = 8.8
                    frames.elementPositionY = 0.7
                    
        elif newDistrict == 'Town of Collbran':

            lblGrandJunction.elementPositionX = 10.0
            lblFruita.elementPositionX = 10.0
            lblCollbran.elementPositionX = 8.27
            lblCollbran.elementPositionY = 2.63
            lblDebeque.elementPositionX = 10.0
            lblPalisade.elementPositionX = 10.0
            lblCounty.elementPositionX = 12.0

            txtCounty.elementPositionX = 10.0
            txtGrandJunction.elementPositionX = 10.0
            txtFruita.elementPositionX = 10.0
            txtCollbran.elementPositionX = 3.18
            txtCollbran.elementPositionY = 1.07
            txtDebeque.elementPositionX = 10.0
            txtPalisade.elementPositionX = 10.0
        
            for frames in dataframes:
                if frames.name == 'df_Collbran':
                    frames.elementPositionX = 5.7
                    frames.elementPositionY = 0.5
                elif frames.name == 'df_Address':
                    frames.elementPositionX = 0.2745
                    frames.elementPositionY = 2.9452
                else:
                    frames.elementPositionX = 8.8
                    frames.elementPositionY = 0.7
                    
        elif newDistrict == 'Town of De Beque':
            
            lblGrandJunction.elementPositionX = 10.0
            lblFruita.elementPositionX = 10.0
            lblCollbran.elementPositionX = 10.0
            lblDebeque.elementPositionX = 8.27
            lblDebeque.elementPositionY = 2.63
            lblPalisade.elementPositionX = 10.0
            lblCounty.elementPositionX = 12.0

            txtCounty.elementPositionX = 10.0
            txtGrandJunction.elementPositionX = 10.0
            txtFruita.elementPositionX = 10.0
            txtCollbran.elementPositionX = 10.0
            txtDebeque.elementPositionX = 3.18
            txtDebeque.elementPositionY = 1.07
            txtPalisade.elementPositionX = 10.0
            
            for frames in dataframes:
                if frames.name == 'df_Debeque':
                    frames.elementPositionX = 5.7
                    frames.elementPositionY = 0.5
                elif frames.name == 'df_Address':
                    frames.elementPositionX = 0.2745
                    frames.elementPositionY = 2.9452
                else:
                    frames.elementPositionX = 8.8
                    frames.elementPositionY = 0.7

    else:
        lblGrandJunction.elementPositionX = 10.0
        lblFruita.elementPositionX = 10.0
        lblCollbran.elementPositionX = 10.0
        lblDebeque.elementPositionX = 10.0
        lblPalisade.elementPositionX = 10.0
        lblCounty.elementPositionX = 8.27
        lblCounty.elementPositionY = 2.63

        txtCounty.elementPositionX = 3.18
        txtCounty.elementPositionY = 1.07
        txtGrandJunction.elementPositionX = 10.0
        txtFruita.elementPositionX = 10.0
        txtCollbran.elementPositionX = 10.0
        txtDebeque.elementPositionX = 10.0
        txtPalisade.elementPositionX = 10.0

        for frames in dataframes:
            if frames.name == 'df_County':
                frames.elementPositionX = 5.7
                frames.elementPositionY = 0.5
            elif frames.name == 'df_Address':
                frames.elementPositionX = 0.2745
                frames.elementPositionY = 2.9452
            else:
                frames.elementPositionX = 8.8
                frames.elementPositionY = 0.7

    ddp.exportToPDF(pgLabel + ".pdf", "CURRENT")
    finalPdf.appendPages(pgLabel + ".pdf")

    #Clean-up before next page (deletes cellTxt values & extra lines)
    arcpy.Delete_management("in_memory\select1")
    arcpy.Delete_management("in_memory\sort1")
    arcpy.DeleteRows_management(tempTable)
    
    for elm in arcpy.mapping.ListLayoutElements(mxd, "GRAPHIC_ELEMENT", "*clone*"):
        elm.delete()
    for elm in arcpy.mapping.ListLayoutElements(mxd, "TEXT_ELEMENT", "*clone*"):
        elm.delete() 

    cellTxt.elementPositionX = -2
    headerTxt.elementPositionX = -2
    horzline.elementPositionX = -2
    vertline.elementPositionX = -2

# Commit Changes and delete variable references
finalPdf.saveAndClose()
del finalPdf
