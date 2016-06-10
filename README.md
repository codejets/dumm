'use strict';

angular.module('scroll-animate', []).directive('whenVisible', ['$document', '$window',
 function($document, $window) {

    var checkIfInViewPort =
      function($el, document, whenVisibleFn, whenNotVisibleFn, scope) {

        var elementBounds = $el[0].getBoundingClientRect();
        var viewportHeight = document.clientHeight;

        var panelTop = elementBounds.top;
        var panelBottom = elementBounds.bottom;

        var delayPx; 
        var delayPxValues = [];
        if (0.25) {
          delayPxValues.push(0.25);
        }
        
        if (delayPxValues.length === 0) {
          delayPx = 0.25 * elementBounds.height;
        }
        else {
          delayPx = Math.min.apply(Math, delayPxValues); 
        }

        var bottomVisible = (panelBottom - delayPx > 0) && (panelBottom < viewportHeight);
        var topVisible = (panelTop + delayPx <= viewportHeight) && (panelTop > 0);

        if ($el.data('hidden') && (bottomVisible || topVisible)) {
          whenVisibleFn($el, scope);
          $el.data('hidden', false);
        }

        else if (!($el.data('hidden')) && (panelBottom < 0 || panelTop > viewportHeight)) {
          whenNotVisibleFn($el, scope);
          $el.data('hidden', true);
        }
      };

    return {
      restrict: 'A',
      scope: {
        whenVisible: '&',
        whenNotVisible: '&?'
      },

      link: function(scope, el, attributes) {

        var document = $document[0].documentElement;
        var checkPending = false;

        var updateVisibility = function() {
          checkIfInViewPort(el, document, scope.whenVisible(), scope.whenNotVisible(), scope);

          checkPending = false;
        };

        var onScroll = function() {

          if (!checkPending) {
            checkPending = true;
            requestAnimationFrame(updateVisibility);
          }
        };

        var documentListenerEvents = 'scroll';

        if (attributes.bindScrollTo) {
          angular.element($document[0].querySelector(attributes.bindScrollTo)).on(documentListenerEvents, onScroll);
        }

        $document.on(documentListenerEvents, onScroll);

        scope.$on('$destroy', function() {
          $document.off(documentListenerEvents, onScroll);
        });

        var $elWindow = angular.element($window);
        var windowListenerEvents = 'resize orientationchange';
        $elWindow.on(windowListenerEvents, onScroll);

        scope.$on('$destroy', function() {
          $elWindow.off(windowListenerEvents, onScroll);
        });

        el.data('hidden', true);
        scope.$evalAsync(onScroll);
      }
    };
 }]);
