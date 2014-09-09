EV-project
==========

clear;
chdir('C:\Users\bh4045\Desktop\Extensional Viscosity Project'); // name of directory
fid = mopen('33-1.txt','rt'); // name of the text file
ctr=1;
stackno=0; // initialize image no to 0
LenYV=[]; // initiliaze LenYV vector 
MinRV=[]; //initialize MinRV vector
while(~meof(fid)) do
    rowscan=mfscanf(4, fid, '%d');
    if rowscan(3) == stackno then // if the row is for same image then add to imagestack
        imagestack(ctr,:) = rowscan(1:4)';
    else  // if new image then update stackno to make a new stack and analyze the current image
        disp(stackno)
        if stackno <300 then // temporary line to analyze a particular image
            //scf(); plot2d('nn',imagestack(:,1),imagestack(:,2),style=-1); // display current image
            //p = get("hdl"); p.children.mark_mode = "on"; p.children.mark_style = 9; p.children.thickness = 3; p.children.mark_foreground = 1;
            //MinR=min(LenYV);
            UniqueListX = unique(imagestack(:,1)); // determine unique list of X coord
            for j=1:length(UniqueListX) // for each X
                IndexForX = find(imagestack(:,1) == UniqueListX(j)); 
                SortedListY = gsort(imagestack(IndexForX,2),'g','i'); // determine the Y list
//                disp(SortedListY');
                clear EdgeLoc_thick;
                // scan the SortedYList and remove continuous points from calculations and record them in another vector along with starting point
                EdgeLoc_thick(1,:) = [SortedListY(1), 1]; 
                for k=2:length(SortedListY)
                    if SortedListY(k) == SortedListY(k-1)+1 then
                        EdgeLoc_thick($,2) = EdgeLoc_thick($,2)+1;
                    else 
                        EdgeLoc_thick($+1,:) = [SortedListY(k), 1];
                    end
                end
                LenY = 0; // initialize the LenY variable
                if size(EdgeLoc_thick,1)>1 then // if there is more than one "thick edge"
                    LenY = sum(EdgeLoc_thick(2:$,:))-EdgeLoc_thick(1:$-1,1); // determine the thickness taking into accout the continous set
                else
                    LenY = (EdgeLoc_thick(1,2));
                end
//                       // ****************** Write a module to find continous sequence of ones ****************** 
//                    end
//                end
                if j ~= 1 then // If not the first line that is being scanned in an image
                    [Val,Index] = min(abs(LenY - UniqueListX(j-1,2))); // Find the closest number in the previous scan
                    LenY = LenY(Index);
                else 
                    // ****************** Needs a module for a case when the 1st scan has some issue ****************** 
                end

                //disp(LenY);
                UniqueListX(j,2) = LenY;
                
                // write the module to find the local minima
                
                LenYV=[LenYV; LenY];
                
            end  
            //scf(); plot2d('nn',UniqueListX(:,1),UniqueListX(:,2),style=-2);
            //xlabel('X location (pixels)'); ylabel('Thickness (pixels)');
//            p = get("hdl"); p.children.mark_mode = "on"; p.children.mark_style = 9; p.children.thickness = 3; p.children.mark_foreground = 1;
            //break; 
            MinR=min(LenYV);
            MinRV=[MinRV; MinR];
            disp(MinR);
            
        end // part of temp if loop            
        ctr = 1;
        clear imagestack; // reset the imagestack variable
        imagestack(ctr,:) = rowscan(1:4)'; // save the row into the new imagestack varaible
        stackno = rowscan(3); // update the stackno
    end
    ctr=ctr+1;
end
disp(MinRV);
mclose(fid);


