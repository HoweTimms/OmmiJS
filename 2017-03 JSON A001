//Ver 0.06c

// Extended range of JSON that can be diagrammed
// complete now ? 
//Import a JSON file to a nested OG6 diagram
// (JavaScript for Automation)
// Run from Script editor with language selected at top left set to JavaScript
// 1. Choose a .json from the file dialog which appears
// 2. Confirm

// Draws a nested diagram using a specified:
// - Template
// - Width and height for node shapes

// Select alternative template name and/or shape size in last line of script
(function (strTemplate, lstSize) {
    'use strict';


    // CONVERT ARBITRARY JSON TO A SIMPLE {text:String, nest:Array} tree
    // a -> [s, [b, c]]
    function jsonTree(a) {
        
        return isAtom(a) ? {
                text: a !== null ? a.toString() : 'null'
            } : Object.keys(a)
            .reverse()
            .map(function (k) {
                var subTree = jsonTree(a[k]);
                return {
                    text: k,
                    nest: (isArray(subTree) ? subTree : [subTree])
                };
            });
    }

    // [tree] -> tree
    function rootedTree(lstTree) {
        return lstTree.length > 1 ? [{
            text: "",
            nest: lstTree
        }] : lstTree;
    }

    // a -> Bool
    function isAtom(a) {
        return -1 !== [
			'boolean', 'undefined', 'number', 'string'
		].indexOf(typeof a) || null === a;
    }


    // a -> Bool
    function isArray(a) {
        return a instanceof Array;
    }

    // for type strings see: Apple's 'System-Declared Uniform Type Identifiers'
    // if strType is omitted, files of all types will be selectable
    // String -> String -> String
    function pathChoice(strPrompt, strType) {
        var a = Application.currentApplication();

        return (a.includeStandardAdditions = true, a)
            .chooseFile({
                withPrompt: strPrompt,
                ofType: strType
            })
            .toString();
    }

    // String -> String
    function textFileContents(strPath) {
        return $.NSString.stringWithContentsOfFile(strPath);
    }


    // [{}] -> String -> (Integer, Integer) -> OmniGraffle6Doc
    function jsoToOG6(lstJSO, strTemplate, lstSize) {
        function drawTree(graphics, shp, idShape, lstChiln) {

            lstChiln.forEach(function (oNode) {
                var shpChild = og.Shape({
                        text: oNode.text,
                        size: lstSize //,
                            //notes: oNode.note
                    }),
                    lstNest = oNode.nest || []; //
                //dctTags = oNode.tags;

                // add shape to canvas graphics
                graphics.push(shpChild);

                var idChild = shpChild.id();

                // record link for later creation through AppleScript kludge
                // (see connect bug below)
                lstLinks.push([idShape, idChild]);

                // BUG: It seems that the connect command may be broken in the JS interface
                //      (trips a type error)
                // og.connect(shp(), {
                //  to: child()
                // });

                if (lstNest && lstNest.length) {
                    drawTree(graphics, shpChild, idChild,
                        lstNest);
                }

                //if (dctTags) addData(shpChild, dctTags);

            });
        };

        //         function addData(shp, dctUser) {
        //             var userData = shp.userDataItems;
        // 
        //             Object.keys(dctUser).forEach(function (k) {
        //                 userData[k].value = dctUser[k];
        //             });
        //         }



        if (lstJSO.length > 0) {

            var og = Application("OmniGraffle"),
                a = Application.currentApplication(),
                sa = (a.includeStandardAdditions = true, a),
                blnTemplate = og.availableTemplates()
                .map(function (x) {
                    return x.split("/")[1];
                })
                .indexOf(strTemplate) !== -1;


            // Template found ?
            if (!blnTemplate) {
                sa.activate();
                sa.displayDialog('Template not found:\n\n\t"' +
                    strTemplate +
                    '\"', {
                        withTitle: "Import JSON into OmniGraffle"
                    });
                return;
            }

            var ds = og.documents,
                d = ((function () {
                    return ds.push(
                        og.Document({
                            template: strTemplate
                        })
                    )
                })() && ds[0]),
                cnv = d ? d.canvases.at(0) : undefined,
                oLayout = cnv ? cnv.layoutInfo() : undefined;


            if (oLayout) {
                var graphics = cnv.graphics,
                    layer = d.layers.at(0),
                    lstLinks = [];

                // suspended during drawing
                oLayout.automaticLayout = false;
                if (graphics.length) og.delete(graphics);

                lstJSO.forEach(function (oNode) {
                    var shp = og.Shape({
                            text: oNode.text,
                            size: lstSize,
                            notes: oNode.note
                        }),
                        lstChiln = oNode.nest,
                        dctTags = oNode.tags;

                    graphics.push(shp);

                    if (lstChiln) {
                        drawTree(graphics, shp, shp.id(),
                            lstChiln);
                    }

                    if (dctTags) addData(shp, dctTags);
                });

                // FALL BACK TO APPLESCRIPT KLUDGE EMBEDDED IN JS TO CREATE PARENT-CHILD LINKS BETWEEN SHAPES
                // (The JavaScript interface to doing this appears to be broken at the moment)
                sa.doShellScript(
                    "osascript <<AS_END 2>/dev/null\ntell application \"OmniGraffle\" to tell front canvas of front document\n" +
                    "set recP to {line type: straight}\n" +
                    lstLinks.map(function (pair) {
                        return 'connect shape id ' + pair[0] +
                            ' to shape id ' + pair[1] //+
                            //' with properties recP';
                    })
                    .join('\n') + "\nend tell\nAS_END\n"
                );

                // restored
                oLayout.automaticLayout = true;
                cnv.layout(graphics);
                og.activate();

            }
        }
    }


    // MAIN

    jsoToOG6( // Create a basic diagram with specified template and node size
        rootedTree( // Supply a single virtual root if trees are multiple
            jsonTree( // Normalise JSO to diagrammable {text:strText, nest:lstChildren} tree
                JSON.parse(
                    textFileContents(
                        pathChoice(
                            'JSON file:',
                            'public.text'
                        )
                    )
                    .js
                )
            )
        ),
        strTemplate, lstSize
    );

})('Hierarchical', [80, 80]);
