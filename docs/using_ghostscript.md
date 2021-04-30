# Using Ghostscript to edit or maninpulate PDFs

% to slice a page or pages out of a PDF with gs
gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH -dSAFER -dFirstPage=5 -dLastPage=5 -sOutputFile=OUTPUT.pdf INPUT.pdf

% to combine pages into a single PDF with gs
gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH -dSAFER -sOutputFile=combined.pdf first.pdf second.pdf third.pdf

% to extract odd pages only
gs -sDEVICE=pdfwrite -sPageList=odd -sOutputFile=odd.pdf -dBATCH -dNOPAUSE file.pdf


% to rotate - not working !
gs -sDEVICE=pdfwrite -dNOPAUSE -dBATCH -dSAFER -dAutoRotatePages=/All -sOutputFile=OUTPUT.pdf INPUT.pdf

% use pdftk, this works - rotate counterclockwise
 pdftk p1.pdf cat 1-endwest output out.pdf

 To rotate page 1 by 90 degrees clockwise:

Copy
pdftk in.pdf cat 1E output out.pdf    # old pdftk
pdftk in.pdf cat 1east output out.pdf # new pdftk

To rotate all pages clockwise:

Copy
pdftk in.pdf cat 1-endE output out.pdf    # old pdftk
pdftk in.pdf cat 1-endeast output out.pdf # new pdftk%