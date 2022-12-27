# Course Material and FAQ for my Complete JavaScript Course

This repo contains starter files and final code for all sections and projects contained in the course.

Use starter code to start each section, and **final code to compare it with your own code whenever something doesn't work**!

👇 **_Please read the following Frequently Asked Questions (FAQ) carefully before starting the course_** 👇

## FAQ

### Q1: How do I download the files?

**A:** If you're new to GitHub and just want to download the entire code, hit the green button saying "Code", and then choose the "Download ZIP" option. If you can't see the button (on mobile), use [this link](https://github.com/jonasschmedtmann/complete-javascript-course/archive/master.zip) instead.

### Q2: I'm looking for the old course version (v1) files. Where can I find them?

**A:** They are in this same repo, but in the [v1 branch](https://github.com/jonasschmedtmann/complete-javascript-course/tree/v1). So just go to [v1](https://github.com/jonasschmedtmann/complete-javascript-course/tree/v1), and download the code from there.

### Q3: I'm stuck! Where do I get help?

**A:** Have you actually tried to fix the problem on your own? Have you compared your code to the final code? If you failed fixing your problem, please **post a detailed description of the problem to the Q&A area of that video over at Udemy**, along with a [codepen](https://codepen.io/pen/) containing your code. You will get help there. Please don't send me a personal message or email to fix coding problems.

### Q4: What VSCode theme are you using?

**A:** I use Monokai Pro for all my coding and course production. It's a paid theme (I', **not** affiliated with them), but you can actually use the free demo version forever 😅

### Q5: Can I see a final version of the course projects?

**A:** Sure! Here you go:

- [Pig Game](https://pig-game-v2.netlify.app) (DOM Manipulation)
- [Bankist](https://bankist.netlify.app/) (Arrays, Numbers, Dates, Timers. Fake "log in" with user `js` and PIN `1111`)
- [Bankist Site](https://bankist-dom.netlify.app/) (Advanced DOM and Events)
- [Mapty](https://mapty.netlify.app/) (OOP, Geolocation, Project planning)
- [forkify](https://forkify-v2.netlify.app/) (Final advanced project)

### Q6: Videos don't load, can you fix it?

**A:** Unfortunately, there is nothing I can do about it. The course is hosted on Udemy, and sometimes they have technical issues like this. Please just come back a bit later or [contact their support team](https://support.udemy.com/hc/en-us).

### Q7: Videos are blurred / have low quality, can you fix it?

**A:** Please open video settings and change the quality from 'Auto' to another value, for example 720p. If that doesn't help, please [contact the Udemy support team](https://support.udemy.com/hc/en-us).

### Q8: Are the videos downloadable?

**A:** Yes! I made all videos downloadable from Udemy so you can learn even without an internet connection. To download a video, use the settings icon in the right bottom corner of the video player. Videos have to be downloaded individually.

### Q9: I want to put these projects in my portfolio. Is that allowed?

**A:** Absolutely! Just make sure you actually built them yourself by following the course, and that you understand what you did. What is **not allowed** is that you create your own course/videos/articles based on this course's content!

### Q10: You mentioned your resources page. Where can I find it?

**A:** It's on my website at <http://codingheroes.io/resources>. You can subscribe for updates 😉

### Q11: I love your courses and want to get updates on new courses. How?

**A:** First, you can subscribe to my email list [at my website](http://codingheroes.io/resources). Plus, I make important announcements on twitter [@jonasschmedtman](https://twitter.com/jonasschmedtman), so you should definitely follow me there 🔥

### Q12: How do I get my certificate of completion?

**A:** A certificate of completion is provided by Udemy after you complete 100% of the course. After completing the course, just click on the "Your progress" indicator in the top right-hand corner of the course page. If you want to change your name on the certificate, please [contact the Udemy support team](https://support.udemy.com/hc/en-us).

### Q13: Can you add subtitles in my language?

**A:** No. I provide professional English captions, but Udemy is responsible for subtitles in all other languages (automatic translations). So please [contact the Udemy support team](https://support.udemy.com/hc/en-us) to request your own language.

### Q14: Do you accept pull requests?

**A:** No, for the simple reason that I want this repository to contain the _exact_ same code that is shown in the videos. However, please feel free to add an issue if you found one.


@Library(['pipeline-framework','pipeline-toolbox']) _
env.LOG_LEVEL = "debug"

affectedFiles = [:]

pipeline {
    agent none 
    stages {
		stage('Compute Diff') {
            agent any
            // NOT WORKING AS EXPECTED "Stage "Compute Diff" skipped due to when conditional"
            // when {
            //     changeset "**/*.{json,yaml,yml}"
            // }
            steps {
                script {
                  	def files = getPrChangedFiles()
                  	println(files)
    				def otherFilesAffected = false
            		for (int k = 0; k < files.size(); k++) {
                		def file = files[k]
                      	println(file)
                		if (file =~ /\.(ya?ml|json)$/) {
                    		affectedFiles.put(file, '')
                		}
                		else {
                    		otherFilesAffected = true
                		}
           			}
    				if (otherFilesAffected) {
        				echo "Be aware other kind of files have been modified."
    				}
                    echo "Number of affected files ${affectedFiles.size()}"
                }
            }
        }
		stage('Delete Existing Comments') {
			agent any
			steps {
				script {
					println(isPullRequest())
                    println(env.CHANGE_ID)
                  
                    if (isPullRequest()) {
                        def PR_ID = env.CHANGE_ID
						affectedFiles.each { entry ->
							println(entry.key)
							def listOfComments = httpRequest acceptType: 'APPLICATION_JSON',
										authentication: 'IZ_USER',
										contentType: 'APPLICATION_JSON',
										httpMode: 'GET',
										url: "https://giturl/git/rest/api/latest/projects/$BITBUCKET_PROJECT/repos/$BITBUCKET_REPOSITORY/pull-requests/${PR_ID}/comments?path=${entry.key}",
										wrapAsMultipart: false
						
							println(listOfComments)	
							def LIST_OF_COMMENTS = readJSON text: listOfComments.getContent()
							
						
							// println(LIST_OF_COMMENTS)
							// println(LIST_OF_COMMENTS.values)
						
							LIST_OF_COMMENTS.values.each { commentI ->
								def COMMENT_ID = commentI.id
							
								// println(COMMENT_ID)
								def VERSION = commentI.version
								// println(VERSION)
                          
                    				if(commentI.author.name == "swb2-izu-admg-api") {
                                    	def deleteComment = httpRequest acceptType: 'APPLICATION_JSON',
                                            authentication: 'IZ_USER',
                                            contentType: 'APPLICATION_JSON',
                                            httpMode: 'DELETE',
                                            url: "https://giturl/git/rest/api/latest/projects/$BITBUCKET_PROJECT/repos/$BITBUCKET_REPOSITORY/pull-requests/${PR_ID}/comments/${COMMENT_ID}?version=${VERSION}",
                                            wrapAsMultipart: false
                                	}
                                
							} 
						}	
					}
				}
			}
		}
        stage('Validate Guidelines') {
            agent {
                docker {
                    image 'stoplight-docker/amadeus-spectral-with-git:2.0.0'
                    registryUrl '<artifactory>'
                    // registryCredentialsId 'credentials-id'
                }
            }
            steps {
                script {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'IZ_USER', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]){
                    sh 'git clone -b master https://$USERNAME:$PASSWORD@<giturl>/git/scm/adt-sl/spectral-linter-ruleset-for-scdms.git'
                    }
                      sh "ls -lrt"
                      sh "cd spectral-linter-ruleset-for-scdms"
                      sh "ls -lrt spectral-linter-ruleset-for-scdms"
                      sh "spectral --version"
                    affectedFiles.each { entry ->
                        affectedFiles[entry.key] = sh (
                            script: "spectral lint $entry.key --ruleset './spectral-linter-ruleset-for-scdms/.spectral.yaml' -f json -q || true",
                            returnStdout: true
                        )
                    }
                }
            }
        }
        stage('Comment PR') {
            agent any
            steps {
                script {
                	println(isPullRequest())
                    println(env.CHANGE_ID)
                  
                    if (isPullRequest()) {
                        def PR_ID = env.CHANGE_ID
                        
                        affectedFiles.each { entry ->
                            
                            def spectral_report = readJSON text: entry.value
                            println(spectral_report)
                            spectral_report.each { item ->
                                def severity_word = ""
                                def comment = ""
                                switch(item.severity) { 
                                    case 0: 
                                        severity_word = "Error"
                                    break; 
                                    case 1: 
                                        severity_word = "Warning"
                                    break; 
                                    case 2: 
                                        severity_word = "Info"
                                    break; 
                                    case 3: 
                                        severity_word = "Hint"
                                    break; 
                                    default:
                                        severity_word = "Undefined"
                                    break; 
                                } 
                              message = item.message.replaceAll('"', '')
                              if(message =~ '^FetchError' || message =~ '^ENOENT')
                              {
                              }
                              else{
                                
                                if (item.range.start.line == 0) {
                                    comment = """{ "text": "Line ${item.range.start.line + 1} : **$severity_word** - $message", "anchor": { "path": "$entry.key", "diffType": "EFFECTIVE" }}"""
                                }
                                else {
                                    comment = """{ "text": "Line ${item.range.start.line + 1} : **$severity_word** - $message", "anchor": { "line": ${item.range.start.line + 1}, "lineType": "CONTEXT", "diffType": "EFFECTIVE", "path": "$entry.key" }}"""
                                }
                              

 


                                def comment_response_raw = httpRequest acceptType: 'APPLICATION_JSON',
                                    authentication: 'IZ_USER',
                                    contentType: 'APPLICATION_JSON',
                                    httpMode: 'POST',
                                    requestBody: comment,
                                    url: "https://<giturl>/git/rest/api/latest/projects/$BITBUCKET_PROJECT/repos/$BITBUCKET_REPOSITORY/pull-requests/${PR_ID}/comments",
                                    wrapAsMultipart: false

 


                                sleep time: 500, unit: 'MILLISECONDS'
                              
                            }
                            }
                        }
                    }
                }
            }
        }
    }
}