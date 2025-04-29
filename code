<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Resume Formatter</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.6.0/mammoth.browser.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/PapaParse/5.4.1/papaparse.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/tailwindcss/2.2.19/tailwind.min.css" rel="stylesheet">
</head>
<body class="bg-gray-100 min-h-screen">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, useRef } = React;

    // Main App Component
    const App = () => {
      const [file, setFile] = useState(null);
      const [resumeContent, setResumeContent] = useState(null);
      const [parsedContent, setParsedContent] = useState(null);
      const [isLoading, setIsLoading] = useState(false);
      const [errorMessage, setErrorMessage] = useState("");
      const [formattedResume, setFormattedResume] = useState(null);
      const resumeRef = useRef(null);

      // Handle file selection
      const handleFileChange = (e) => {
        const selectedFile = e.target.files[0];
        if (selectedFile) {
          setFile(selectedFile);
          setErrorMessage("");
          setResumeContent(null);
          setParsedContent(null);
          setFormattedResume(null);
        }
      };

      // Process the uploaded file
      const processFile = async () => {
        if (!file) {
          setErrorMessage("Please select a file first");
          return;
        }

        setIsLoading(true);
        setErrorMessage("");
        
        try {
          const fileType = file.name.split('.').pop().toLowerCase();
          let extractedText = "";
          
          // Handle different file types
          if (fileType === 'docx') {
            extractedText = await processDocx(file);
          } else if (fileType === 'pdf') {
            extractedText = await processPdf(file);
          } else if (['txt', 'md'].includes(fileType)) {
            extractedText = await processTextFile(file);
          } else if (['xlsx', 'xls', 'csv'].includes(fileType)) {
            extractedText = await processSpreadsheet(file, fileType);
          } else {
            throw new Error("Unsupported file format. Please upload a .docx, .pdf, .txt, .md, .xlsx, .xls, or .csv file.");
          }
          
          setResumeContent(extractedText);
          
          // Parse the content into structured data
          const parsed = parseResumeContent(extractedText);
          setParsedContent(parsed);
          
          // Format the resume with the parsed content
          const formatted = formatResume(parsed);
          setFormattedResume(formatted);
          
        } catch (error) {
          console.error("Error processing file:", error);
          setErrorMessage(error.message || "Failed to process the file. Please try another one.");
        } finally {
          setIsLoading(false);
        }
      };

      // Process DOCX files
      const processDocx = async (file) => {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = function(event) {
            const arrayBuffer = event.target.result;
            
            mammoth.extractRawText({ arrayBuffer })
              .then(result => {
                resolve(result.value);
              })
              .catch(error => {
                reject(new Error("Failed to process DOCX file: " + error.message));
              });
          };
          reader.onerror = () => reject(new Error("Failed to read the file"));
          reader.readAsArrayBuffer(file);
        });
      };

      // Process PDF files
      const processPdf = async (file) => {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = async function(event) {
            const arrayBuffer = event.target.result;
            try {
              // Set worker path for PDF.js
              pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
              
              const loadingTask = pdfjsLib.getDocument(arrayBuffer);
              const pdf = await loadingTask.promise;
              
              let fullText = '';
              for (let i = 1; i <= pdf.numPages; i++) {
                const page = await pdf.getPage(i);
                const textContent = await page.getTextContent();
                const pageText = textContent.items.map(item => item.str).join(' ');
                fullText += pageText + '\n';
              }
              
              resolve(fullText);
            } catch (error) {
              reject(new Error("Failed to process PDF file: " + error.message));
            }
          };
          reader.onerror = () => reject(new Error("Failed to read the file"));
          reader.readAsArrayBuffer(file);
        });
      };

      // Process text files
      const processTextFile = async (file) => {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = function(event) {
            resolve(event.target.result);
          };
          reader.onerror = () => reject(new Error("Failed to read the text file"));
          reader.readAsText(file);
        });
      };

      // Process spreadsheet files
      const processSpreadsheet = async (file, fileType) => {
        return new Promise((resolve, reject) => {
          const reader = new FileReader();
          reader.onload = function(event) {
            try {
              let text = '';
              
              if (fileType === 'csv') {
                // Parse CSV
                const result = Papa.parse(event.target.result, {
                  header: true,
                  skipEmptyLines: true
                });
                
                if (result.data && result.data.length) {
                  // Convert CSV data to text
                  Object.entries(result.data[0]).forEach(([key, value]) => {
                    text += `${key}: ${value}\n`;
                  });
                }
              } else {
                // Parse Excel file
                const data = new Uint8Array(event.target.result);
                const workbook = XLSX.read(data, { type: 'array' });
                
                // Get first worksheet
                const firstSheet = workbook.Sheets[workbook.SheetNames[0]];
                
                // Convert to JSON
                const jsonData = XLSX.utils.sheet_to_json(firstSheet);
                
                if (jsonData.length > 0) {
                  // Format as text
                  Object.entries(jsonData[0]).forEach(([key, value]) => {
                    text += `${key}: ${value}\n`;
                  });
                }
              }
              
              resolve(text || "No data found in the spreadsheet");
            } catch (error) {
              reject(new Error("Failed to process spreadsheet: " + error.message));
            }
          };
          
          if (fileType === 'csv') {
            reader.readAsText(file);
          } else {
            reader.readAsArrayBuffer(file);
          }
        });
      };

      // Parse resume content into structured data
      const parseResumeContent = (text) => {
        // Enhanced parser that matches the specific template format
        const parsed = {
          name: "",
          tagline: "",
          contact: {
            email: "",
            phone: "",
            linkedin: ""
          },
          summary: "",
          skills: [],
          experience: [],
          education: [],
          certifications: []
        };
        
        // Extract name (assuming it's the first line)
        const lines = text.split('\n').filter(line => line.trim());
        if (lines.length > 0) {
          parsed.name = lines[0].trim();
        }
        
        // Try to extract tagline (often the second line or after the name)
        if (lines.length > 1) {
          const possibleTaglines = lines.slice(1, 3);
          const taglineCandidate = possibleTaglines.find(line => 
            !line.includes('@') && 
            !line.match(/^\d/) && 
            !line.includes('linkedin') &&
            line.length < 100
          );
          if (taglineCandidate) {
            parsed.tagline = taglineCandidate.trim();
          }
        }
        
        // Extract email using regex
        const emailRegex = /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b/g;
        const emailMatches = text.match(emailRegex);
        if (emailMatches && emailMatches.length > 0) {
          parsed.contact.email = emailMatches[0];
        }
        
        // Extract phone number
        const phoneRegex = /(\+\d{1,3}|\b)[- ]?(\(?\d{3}\)?)[- ]?(\d{3})[- ]?(\d{4})\b/g;
        const phoneMatches = text.match(phoneRegex);
        if (phoneMatches && phoneMatches.length > 0) {
          parsed.contact.phone = phoneMatches[0];
        }
        
        // Extract LinkedIn (improved approach)
        const linkedinRegex = /(?:linkedin\.com\/in\/[a-zA-Z0-9-]+|linkedin:\s*[a-zA-Z0-9-]+)/gi;
        const linkedinMatches = text.match(linkedinRegex);
        if (linkedinMatches && linkedinMatches.length > 0) {
          parsed.contact.linkedin = linkedinMatches[0];
        }

        // Try to extract sections more thoroughly based on the template format
        
        // Extract PROFESSIONAL SUMMARY
        const summaryRegex = /(?:PROFESSIONAL\s+SUMMARY|SUMMARY|PROFILE|OBJECTIVE|ABOUT)(?:[\s\S]*?)(?=EDUCATION|EXPERIENCE|WORK|SKILLS|CERTIFICATIONS|$)/i;
        const summaryMatch = text.match(summaryRegex);
        if (summaryMatch) {
          // Remove the heading and clean up
          parsed.summary = summaryMatch[0].replace(/(?:PROFESSIONAL\s+SUMMARY|SUMMARY|PROFILE|OBJECTIVE|ABOUT)[\s:]*/i, '').trim();
        }
        
        // Extract EDUCATION
        const educationRegex = /(?:EDUCATION)(?:[\s\S]*?)(?=CERTIFICATIONS|EXPERIENCE|WORK|SKILLS|$)/i;
        const educationMatch = text.match(educationRegex);
        if (educationMatch) {
          // Remove the heading
          const educationText = educationMatch[0].replace(/(?:EDUCATION)[\s:]*/i, '').trim();
          
          // Look for degree and institution patterns
          const degreeMatches = educationText.match(/(?:^|\n)([^,\n]+?)(?:\n|$)/g);
          const instituteMatches = educationText.match(/(?:^|\n)([^,\n]*(?:University|College|Institute|School)[^,\n]*)/gi);
          const gpaMatches = educationText.match(/(?:CGPA|GPA)[\s:]*([0-9.]+%?)/i);
          
          if (degreeMatches && degreeMatches.length > 0) {
            parsed.education.push({
              degree: degreeMatches[0].trim(),
              institution: instituteMatches ? instituteMatches[0].trim() : "",
              gpa: gpaMatches ? gpaMatches[1].trim() : ""
            });
          }
        }
        
        // Extract CERTIFICATIONS
        const certificationsRegex = /(?:CERTIFICATIONS)(?:[\s\S]*?)(?=EXPERIENCE|WORK|SKILLS|$)/i;
        const certificationsMatch = text.match(certificationsRegex);
        if (certificationsMatch) {
          // Remove the heading
          const certText = certificationsMatch[0].replace(/(?:CERTIFICATIONS)[\s:]*/i, '').trim();
          
          // Split into potential certification blocks
          const certBlocks = certText.split(/\n\s*\n/);
          
          certBlocks.forEach(block => {
            const lines = block.split('\n').filter(l => l.trim());
            if (lines.length > 0) {
              parsed.certifications.push({
                name: lines[0].trim(),
                issuer: lines.length > 1 ? lines[1].trim() : "",
                date: lines.length > 2 ? lines[2].trim() : ""
              });
            }
          });
        }
        
        // Extract TECH SKILLS
        const skillsRegex = /(?:TECH\s+SKILLS|SKILLS|TECHNOLOGIES|COMPETENCIES)(?:[\s\S]*?)(?=EXPERIENCE|WORK|PROFESSIONAL|$)/i;
        const skillsMatch = text.match(skillsRegex);
        if (skillsMatch) {
          // Remove the heading
          const skillsText = skillsMatch[0].replace(/(?:TECH\s+SKILLS|SKILLS|TECHNOLOGIES|COMPETENCIES)[\s:]*/i, '').trim();
          
          // Try to detect category format: "Category: Skill1, Skill2"
          const categoryMatches = skillsText.match(/([^:,\n]+):\s*([^:]+?)(?=\n[^:,\n]+:|$)/g);
          
          if (categoryMatches && categoryMatches.length > 0) {
            // Skills with categories
            categoryMatches.forEach(match => {
              const [category, skillsList] = match.split(':').map(s => s.trim());
              parsed.skills.push({
                category: category,
                items: skillsList.split(/[,•]/).map(s => s.trim()).filter(Boolean)
              });
            });
          } else {
            // Just a flat list of skills
            parsed.skills = skillsText.split(/[,•\n]/).map(skill => skill.trim()).filter(Boolean);
          }
        }
        
        // Extract PROFESSIONAL EXPERIENCE
        const experienceRegex = /(?:PROFESSIONAL\s+EXPERIENCE|EXPERIENCE|WORK)(?:[\s\S]*?)(?=$)/i;
        const experienceMatch = text.match(experienceRegex);
        if (experienceMatch) {
          // Remove the heading
          const expText = experienceMatch[0].replace(/(?:PROFESSIONAL\s+EXPERIENCE|EXPERIENCE|WORK)[\s:]*/i, '').trim();
          
          // Split into potential job blocks
          const expBlocks = expText.split(/\n\s*\n/);
          
          expBlocks.forEach(block => {
            if (block.trim()) {
              parsed.experience.push({
                description: block.trim()
              });
            }
          });
        }
        
        return parsed;
      };

      // Format the resume with the parsed data
      const formatResume = (data) => {
        if (!data) return null;
        
        return {
          ...data,
          // Add any additional formatting or transformation here
        };
      };

      // Generate PDF from the formatted resume
      const generatePDF = async () => {
        if (!resumeRef.current) return;
        
        const { jsPDF } = window.jspdf;
        
        try {
          setIsLoading(true);
          const canvas = await html2canvas(resumeRef.current, {
            scale: 2,
            logging: false,
            useCORS: true
          });
          
          const imgData = canvas.toDataURL('image/png');
          const pdf = new jsPDF('p', 'mm', 'a4');
          const pdfWidth = pdf.internal.pageSize.getWidth();
          const pdfHeight = pdf.internal.pageSize.getHeight();
          const imgWidth = canvas.width;
          const imgHeight = canvas.height;
          const ratio = Math.min(pdfWidth / imgWidth, pdfHeight / imgHeight);
          const imgX = (pdfWidth - imgWidth * ratio) / 2;
          const imgY = 30;
          
          pdf.addImage(imgData, 'PNG', imgX, imgY, imgWidth * ratio, imgHeight * ratio);
          pdf.save('formatted-resume.pdf');
        } catch (error) {
          console.error("Error generating PDF:", error);
          setErrorMessage("Failed to generate PDF. Please try again.");
        } finally {
          setIsLoading(false);
        }
      };

      return (
        <div className="container mx-auto px-4 py-8">
          <header className="text-center mb-10">
            <h1 className="text-3xl font-bold text-blue-600">Resume Formatter</h1>
                            <p className="text-gray-600 mt-2">Upload your resume to reformat it to a standardized professional template</p>
          </header>

          <div className="bg-white rounded-lg shadow-lg p-6 max-w-4xl mx-auto">
            {/* File Upload */}
            <div className="mb-6">
              <label className="block text-gray-700 text-sm font-bold mb-2">
                Upload Your Resume
              </label>
              <div className="flex items-center">
                <input
                  type="file"
                  onChange={handleFileChange}
                  className="text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
                  accept=".docx,.pdf,.txt,.md,.csv,.xlsx,.xls"
                />
                <button
                  onClick={processFile}
                  disabled={!file || isLoading}
                  className={`py-2 px-4 rounded font-medium ${
                    !file || isLoading
                      ? 'bg-gray-300 text-gray-500 cursor-not-allowed'
                      : 'bg-blue-600 text-white hover:bg-blue-700'
                  }`}
                >
                  {isLoading ? 'Processing...' : 'Process Resume'}
                </button>
              </div>
              {file && <p className="mt-2 text-sm text-gray-500">Selected file: {file.name}</p>}
              {errorMessage && <p className="mt-2 text-sm text-red-500">{errorMessage}</p>}
            </div>

            {/* Results Section */}
            {formattedResume && (
              <div className="mt-8">
                <h2 className="text-xl font-semibold mb-4">Your Formatted Resume</h2>
                
                <div className="border rounded-lg p-8 bg-white shadow" ref={resumeRef}>
                  {/* Formatted Resume Template Following User's Specific Format */}
                  {/* Name - H1 Bold Centered */}
                  <h1 className="text-3xl font-bold text-center">{formattedResume.name || "FirstName LastName"}</h1>
                  
                  {/* Tagline - H2 Bold Centered */}
                  <h2 className="text-xl font-bold text-center mt-2">{formattedResume.tagline || "Professional Tagline"}</h2>
                  
                  {/* Contact Info - H2 Bold Centered */}
                  <h2 className="text-xl font-bold text-center mt-2">
                    {formattedResume.contact.phone || "Phone number"} | {formattedResume.contact.email || "Email address"} | {formattedResume.contact.linkedin || "LinkedIn"}
                  </h2>
                  
                  {/* Horizontal Line */}
                  <h2 className="text-xl text-center mt-2 border-b-2 border-gray-400">________________________________________________________________</h2>
                  
                  {/* PROFESSIONAL SUMMARY */}
                  <div className="mt-6">
                    <h2 className="text-xl font-bold">PROFESSIONAL SUMMARY</h2>
                    <p className="mt-2">{formattedResume.summary || "Sample of professional summary"}</p>
                  </div>
                  
                  {/* Line Space */}
                  <div className="mt-6"></div>
                  
                  {/* EDUCATION */}
                  <div>
                    <h2 className="text-xl font-bold">EDUCATION</h2>
                    {formattedResume.education && formattedResume.education.length > 0 ? (
                      formattedResume.education.map((edu, index) => (
                        <div key={index} className="mt-2">
                          <p className="font-bold">{edu.degree || "Degree name"}</p>
                          <p className="font-bold">{edu.institution || "Institution name"}</p>
                          <p>CGPA: {edu.gpa || "X%"}</p>
                        </div>
                      ))
                    ) : (
                      <div className="mt-2">
                        <p className="font-bold">Degree name</p>
                        <p className="font-bold">Institution name</p>
                        <p>CGPA: X%</p>
                      </div>
                    )}
                  </div>
                  
                  {/* Line Space */}
                  <div className="mt-6"></div>
                  
                  {/* CERTIFICATIONS */}
                  <div>
                    <h2 className="text-xl font-bold">CERTIFICATIONS</h2>
                    {formattedResume.certifications && formattedResume.certifications.length > 0 ? (
                      formattedResume.certifications.map((cert, index) => (
                        <div key={index} className="mt-2">
                          <p className="font-bold">{cert.name || "Certification name"}</p>
                          <p className="font-bold">{cert.issuer || "Issuing institution"}</p>
                          <p className="font-bold">{cert.date || "Date of certification"}</p>
                        </div>
                      ))
                    ) : (
                      <div className="mt-2">
                        <p className="font-bold">Certification name</p>
                        <p className="font-bold">Issuing institution</p>
                        <p className="font-bold">Date of certification</p>
                      </div>
                    )}
                  </div>
                  
                  {/* Line Space */}
                  <div className="mt-6"></div>
                  
                  {/* TECH SKILLS */}
                  <div>
                    <h2 className="text-xl font-bold">TECH SKILLS</h2>
                    {formattedResume.skills && formattedResume.skills.length > 0 ? (
                      <p className="mt-2">
                        {formattedResume.skills.map((skill, index, array) => {
                          // Group skills by category if possible
                          if (typeof skill === 'object' && skill.category && skill.items) {
                            return `${skill.category}: ${skill.items.join(', ')}`;
                          } else {
                            // If no categories, just list all skills
                            return array.join(', ');
                          }
                        })}
                      </p>
                    ) : (
                      <p className="mt-2">Category: Skills</p>
                    )}
                  </div>
                  
                  {/* PROFESSIONAL EXPERIENCE */}
                  <div className="mt-6">
                    <h2 className="text-xl font-bold">PROFESSIONAL EXPERIENCE</h2>
                    {formattedResume.experience && formattedResume.experience.length > 0 ? (
                      formattedResume.experience.map((exp, index) => (
                        <p key={index} className="mt-2">{exp.description || "Sample of professional experience"}</p>
                      ))
                    ) : (
                      <p className="mt-2">Sample of professional experience</p>
                    )}
                  </div>
                  
                  {/* Line Space */}
                  <div className="mt-6"></div>
                </div>
                
                <div className="mt-6 flex justify-center">
                  <button
                    onClick={generatePDF}
                    disabled={isLoading}
                    className={`py-2 px-6 rounded-md font-medium ${
                      isLoading
                        ? 'bg-gray-300 text-gray-500 cursor-not-allowed'
                        : 'bg-green-600 text-white hover:bg-green-700'
                    }`}
                  >
                    {isLoading ? 'Generating...' : 'Download as PDF'}
                  </button>
                </div>
              </div>
            )}
          </div>
          
          <footer className="mt-10 text-center text-sm text-gray-500">
            <p>© {new Date().getFullYear()} Resume Formatter. All rights reserved.</p>
            <p className="mt-1">Free to use for everyone. Hosted on Netlify.</p>
          </footer>
        </div>
      );
    };

    // Render the app
    ReactDOM.render(<App />, document.getElementById('root'));
  </script>
</body>
</html>
