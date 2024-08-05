# cyberball

# Usage in qualtrics

# HTML view
<iframe id="cyberball" width="100%" height="580" src="https://github.com/sabinaraja/cyberball/blob/main/README.md"></iframe>

# Javascript
var throwLog = [];
var leaveButtonText = 'Leave Game';
var maxThrows = 30; // Set the maximum number of throws
var currentThrows = 0; // Counter for the number of throws

Qualtrics.SurveyEngine.addOnload(function() {
    /*Place your JavaScript here to run when the page loads*/
    this.hideNextButton();
});

Qualtrics.SurveyEngine.addOnReady(function() {
    /*Place your JavaScript here to run when the page is fully displayed*/
    var survey = this;

    window.addEventListener('message', function(e) {
        switch(e.data.type) {
            case 'throw':
                throwLog.push(e.data);
                currentThrows++; // Increment throw counter
                checkThrowLimit(); // Check if the throw limit is reached
                break;
            case 'leave':
                throwLog.push(e.data);
                break;
            case 'player-may-leave':
                throwLog.push(e.data);

                jQuery('#NextButton').clone()
                    .attr('disabled', false)
                    .attr('title', leaveButtonText)
                    .attr('value', leaveButtonText)
                    .attr('aria-label', leaveButtonText)
                    .attr('style', 'margin-right: 10px')
                    .prependTo('#Buttons')
                    .one('click', function() {
                        throwLog.push({
                            type: 'player-left'
                        });

                        Qualtrics.SurveyEngine.setEmbeddedData('GameLog', JSON.stringify(throwLog));

                        survey.clickNextButton();
                    });

                break;
            case 'game-end':
                Qualtrics.SurveyEngine.setEmbeddedData('GameLog', JSON.stringify(throwLog));

                survey.showNextButton();

                // Auto-advance?
                //survey.clickNextButton();
                break;
            default:
        }
    });

    // Function to check if the throw limit is reached
    function checkThrowLimit() {
        if (currentThrows >= maxThrows) {
            // End the game when the throw limit is reached
            window.postMessage({ type: 'game-end' }, '*');
        }
    }
});

Qualtrics.SurveyEngine.addOnUnload(function() {

});
