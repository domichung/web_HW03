angular.module("ui.bootstrap", ["ui.bootstrap.tpls", "ui.bootstrap.modal", "ui.bootstrap.pagination"]);
angular.module("ui.bootstrap.tpls", ["template/modal/backdrop.html", "template/modal/window.html", "template/pagination/pager.html", "template/pagination/pagination.html"]);
angular.module('ui.bootstrap.modal', [])

/**
* A helper, internal data structure that acts as a map but also allows getting / removing
* elements in the LIFO order
*/
  .factory('$$stackedMap', function()
  {
      return {
          createNew: function()
          {
              var stack = [];

              return {
                  add: function(key, value)
                  {
                      stack.push({
                          key: key,
                          value: value
                      });
                  },
                  get: function(key)
                  {
                      for (var i = 0; i < stack.length; i++)
                      {
                          if (key == stack[i].key)
                          {
                              return stack[i];
                          }
                      }
                  },
                  keys: function()
                  {
                      var keys = [];
                      for (var i = 0; i < stack.length; i++)
                      {
                          keys.push(stack[i].key);
                      }
                      return keys;
                  },
                  top: function()
                  {
                      return stack[stack.length - 1];
                  },
                  remove: function(key)
                  {
                      var idx = -1;
                      for (var i = 0; i < stack.length; i++)
                      {
                          if (key == stack[i].key)
                          {
                              idx = i;
                              break;
                          }
                      }
                      return stack.splice(idx, 1)[0];
                  },
                  removeTop: function()
                  {
                      return stack.splice(stack.length - 1, 1)[0];
                  },
                  length: function()
                  {
                      return stack.length;
                  }
              };
          }
      };
  })

/**
* A helper directive for the $modal service. It creates a backdrop element.
*/
  .directive('modalBackdrop', ['$modalStack', '$timeout', '$templateCache', function($modalStack, $timeout, $templateCache)
  {
      return {
          restrict: 'EA',
          replace: true,
          //templateUrl: 'template/modal/backdrop.html',
          template: $templateCache.get('template/modal/backdrop.html'),
          link: function(scope, element, attrs)
          {

              //trigger CSS transitions
              $timeout(function()
              {
                  scope.animate = true;
              });

              scope.close = function(evt)
              {
                  var modal = $modalStack.getTop();
                  if (modal && modal.value.backdrop && modal.value.backdrop != 'static')
                  {
                      evt.preventDefault();
                      evt.stopPropagation();
                      $modalStack.dismiss(modal.key, 'backdrop click');
                  }
              };
          }
      };
  } ])

  .directive('modalWindow', ['$modalStack', '$timeout', '$templateCache', function($modalStack, $timeout, $templateCache)
  {
      return {
          restrict: 'EA',
          scope: {
              index: '@'
          },
          replace: true,
          transclude: true,
          //templateUrl: 'template/modal/window.html',
          template: $templateCache.get('template/modal/window.html'),
          link: function(scope, element, attrs)
          {
              scope.windowClass = attrs.windowClass || '';

              //trigger CSS transitions
              $timeout(function()
              {
                  scope.animate = true;
              },200); //fix by jutlin

              scope.close = function(evt)
              {

                  var modal = $modalStack.getTop();
                  if (modal && modal.value.backdrop && modal.value.backdrop != 'static')
                  {
                      evt.preventDefault();
                      evt.stopPropagation();
                      $modalStack.dismiss(modal.key, 'backdrop click');
                  }
              };
          }
      };
  } ])

  .factory('$modalStack', ['$document', '$compile', '$rootScope', '$$stackedMap',
    function($document, $compile, $rootScope, $$stackedMap)
    {

        var backdropjqLiteEl, backdropDomEl;
        var backdropScope = $rootScope.$new(true);
        var html = $document.find('html').eq(0);
        var body = $document.find('body').eq(0);
        var openedWindows = $$stackedMap.createNew();
        var $modalStack = {};

        //        var mainContainer = document.getElementById('mainContainer');
        //        var $mainContainer = angular.element(mainContainer);

        var bodyposition = 0; //addby jutlin
        var bodyIsOverflowing = false;
        var scrollbarWidth = 0;
        var originalBodyPad = null;

        function backdropIndex()
        {
            var topBackdropIndex = -1;
            var opened = openedWindows.keys();
            for (var i = 0; i < opened.length; i++)
            {
                if (openedWindows.get(opened[i]).value.backdrop)
                {
                    topBackdropIndex = i;
                }
            }
            return topBackdropIndex;
        }

        function checkScrollbar()
        {
            var fullWindowWidth = window.innerWidth;
            if (!fullWindowWidth)
            { // workaround for missing window.innerWidth in IE8
                var documentElementRect = document.documentElement.getBoundingClientRect();
                fullWindowWidth = documentElementRect.right - Math.abs(documentElementRect.left);
            }
            bodyIsOverflowing = document.body.clientWidth < fullWindowWidth;
            scrollbarWidth = measureScrollbar();
            
        }

        function measureScrollbar()
        { // thx walsh

            var scrollDiv = document.createElement('div');
            scrollDiv.className = 'modal-scrollbar-measure';
            body.append(scrollDiv);
            var scrollbarWidth = scrollDiv.offsetWidth - scrollDiv.clientWidth;
            body[0].removeChild(scrollDiv);
            return scrollbarWidth;
        }

        function setScrollbar()
        {
            var bodyPad = parseInt((body.css('padding-right') || 0), 10);
            originalBodyPad = document.body.style.paddingRight || '';
            if (bodyIsOverflowing) body.css('padding-right', bodyPad + scrollbarWidth + 'px');

        }

        function resetScrollbar()
        {
            body.css('padding-right', originalBodyPad ? originalBodyPad + 'px' : '');
        }


        $rootScope.$watch(backdropIndex, function(newBackdropIndex)
        {
            backdropScope.index = newBackdropIndex;
        });

        function removeModalWindow(modalInstance)
        {
            
            if (openedWindows.length() == 1) //addby jutlin
            {
                resetScrollbar();
            
                body.css('top', '');
                body.removeClass('modal-open');
                html.removeClass('modal-open');
                //window.scrollTo(0, bodyposition); 

                //            $mainContainer.css('top', ''); 
                //            $mainContainer.removeClass('modal-open'); 
                if (!!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/))
                {
                    setTimeout(function() { parent.scrollTo(0, bodyposition); }, 50);
                }
                else
                {
                    window.scrollTo(0, bodyposition);
                }
            }

            var modalWindow = openedWindows.get(modalInstance).value;

            //clean up the stack
            openedWindows.remove(modalInstance);

            //remove window DOM element
            modalWindow.modalDomEl.remove();

            //remove backdrop if no longer needed
            if (backdropDomEl && backdropIndex() == -1)
            {
                backdropDomEl.remove();
                backdropDomEl = undefined;
            }

            //destroy scope
            modalWindow.modalScope.$destroy();

        }

        $document.bind('keydown', function(evt)
        {
            var modal;

            if (evt.which === 27)
            {
                modal = openedWindows.top();
                if (modal && modal.value.keyboard)
                {
                    $rootScope.$apply(function()
                    {
                        $modalStack.dismiss(modal.key);
                    });
                }
            }
        });




        $modalStack.open = function(modalInstance, modal)
        {


            openedWindows.add(modalInstance, {
                deferred: modal.deferred,
                modalScope: modal.scope,
                backdrop: modal.backdrop,
                keyboard: modal.keyboard
            });

            var angularDomEl = angular.element('<div modal-window></div>');
            angularDomEl.attr('window-class', modal.windowClass);
            angularDomEl.attr('index', openedWindows.length() - 1);
            angularDomEl.html(modal.content);

            var modalDomEl = $compile(angularDomEl)(modal.scope);
            openedWindows.top().value.modalDomEl = modalDomEl;
            body.append(modalDomEl);

            if (backdropIndex() >= 0 && !backdropDomEl)
            {
                backdropjqLiteEl = angular.element('<div modal-backdrop></div>');
                backdropDomEl = $compile(backdropjqLiteEl)(backdropScope);
                body.append(backdropDomEl);
            }

            if (openedWindows.length() == 1) //addby jutlin
            {
                checkScrollbar();
                setScrollbar();


                //bodyposition = body[0].scrollTop; 
                //body.css('top', '-' + bodyposition + 'px'); 
                //body.addClass('modal-open'); 
                //html.addClass('modal-open'); 
                //bodyposition = document.documentElement.scrollTop + body[0].scrollTop;                   
                bodyposition = (document.documentElement && document.documentElement.scrollTop) || document.body.scrollTop; ;

                if (!!navigator.userAgent.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/))
                {//for ios
                    bodyposition = parent.pageYOffset;
                    setTimeout(function()
                    {
                        //$mainContainer.css('top', '-' + bodyposition + 'px');
                        body.css('top', '-' + bodyposition + 'px');
                    }, 50);
                }
                else
                {
                    //$mainContainer.css('top', '-' + bodyposition + 'px');
                    body.css('top', '-' + bodyposition + 'px');
                }

                //$mainContainer.addClass('modal-open'); 
                body.addClass('modal-open');
                html.addClass('modal-open');
            }

        };

        $modalStack.close = function(modalInstance, result)
        {
            var modal = openedWindows.get(modalInstance);
            if (modal)
            {

                modal.value.deferred.resolve(result);
                removeModalWindow(modalInstance);
            }
        };

        $modalStack.dismiss = function(modalInstance, reason)
        {
            var modalWindow = openedWindows.get(modalInstance).value;
            if (modalWindow)
            {

                modalWindow.deferred.reject(reason);
                removeModalWindow(modalInstance);
            }
        };

        $modalStack.getTop = function()
        {
            return openedWindows.top();
        };

        return $modalStack;
    } ])

  .provider('$modal', function()
  {

      var $modalProvider = {
          options: {
              backdrop: true, //can be also false or 'static'
              keyboard: true
          },
          $get: ['$injector', '$rootScope', '$q', '$http', '$templateCache', '$controller', '$modalStack',
        function($injector, $rootScope, $q, $http, $templateCache, $controller, $modalStack)
        {

            var $modal = {};

            function getTemplatePromise(options)
            {
                return options.template ? $q.when(options.template) :
              $http.get(options.templateUrl, { cache: $templateCache }).then(function(result)
              {
                  return result.data;
              });
            }

            function getResolvePromises(resolves)
            {
                var promisesArr = [];
                angular.forEach(resolves, function(value, key)
                {
                    if (angular.isFunction(value) || angular.isArray(value))
                    {
                        promisesArr.push($q.when($injector.invoke(value)));
                    }
                });
                return promisesArr;
            }

            $modal.open = function(modalOptions)
            {

                var modalResultDeferred = $q.defer();
                var modalOpenedDeferred = $q.defer();

                //prepare an instance of a modal to be injected into controllers and returned to a caller
                var modalInstance = {
                    result: modalResultDeferred.promise,
                    opened: modalOpenedDeferred.promise,
                    close: function(result)
                    {
                        $modalStack.close(modalInstance, result);
                    },
                    dismiss: function(reason)
                    {
                        $modalStack.dismiss(modalInstance, reason);
                    }
                };

                //merge and clean up options
                modalOptions = angular.extend({}, $modalProvider.options, modalOptions);
                modalOptions.resolve = modalOptions.resolve || {};

                //verify options
                if (!modalOptions.template && !modalOptions.templateUrl)
                {
                    throw new Error('One of template or templateUrl options is required.');
                }

                var templateAndResolvePromise =
              $q.all([getTemplatePromise(modalOptions)].concat(getResolvePromises(modalOptions.resolve)));


                templateAndResolvePromise.then(function resolveSuccess(tplAndVars)
                {

                    var modalScope = (modalOptions.scope || $rootScope).$new();
                    modalScope.$close = modalInstance.close;
                    modalScope.$dismiss = modalInstance.dismiss;

                    var ctrlInstance, ctrlLocals = {};
                    var resolveIter = 1;

                    //controllers
                    if (modalOptions.controller)
                    {
                        ctrlLocals.$scope = modalScope;
                        ctrlLocals.$modalInstance = modalInstance;
                        angular.forEach(modalOptions.resolve, function(value, key)
                        {
                            ctrlLocals[key] = tplAndVars[resolveIter++];
                        });

                        ctrlInstance = $controller(modalOptions.controller, ctrlLocals);
                    }

                    $modalStack.open(modalInstance, {
                        scope: modalScope,
                        deferred: modalResultDeferred,
                        content: tplAndVars[0],
                        backdrop: modalOptions.backdrop,
                        keyboard: modalOptions.keyboard,
                        windowClass: modalOptions.windowClass
                    });

                }, function resolveError(reason)
                {
                    modalResultDeferred.reject(reason);
                });

                templateAndResolvePromise.then(function()
                {
                    modalOpenedDeferred.resolve(true);
                }, function()
                {
                    modalOpenedDeferred.reject(false);
                });

                return modalInstance;
            };

            return $modal;
        } ]
      };

      return $modalProvider;
  });

angular.module('ui.bootstrap.pagination', [])

.controller('PaginationController', ['$scope', '$attrs', '$parse', '$interpolate', function($scope, $attrs, $parse, $interpolate)
{
    var self = this,
      setNumPages = $attrs.numPages ? $parse($attrs.numPages).assign : angular.noop;

    this.init = function(defaultItemsPerPage)
    {
        if ($attrs.itemsPerPage)
        {
            $scope.$parent.$watch($parse($attrs.itemsPerPage), function(value)
            {
                self.itemsPerPage = parseInt(value, 10);
                $scope.totalPages = self.calculateTotalPages();
            });
        } else
        {
            this.itemsPerPage = defaultItemsPerPage;
        }
    };

    this.noPrevious = function()
    {
        return this.page === 1;
    };
    this.noNext = function()
    {
        return this.page === $scope.totalPages;
    };

    this.isActive = function(page)
    {
        return this.page === page;
    };

    this.calculateTotalPages = function()
    {
        var totalPages = this.itemsPerPage < 1 ? 1 : Math.ceil($scope.totalItems / this.itemsPerPage);
        return Math.max(totalPages || 0, 1);
    };

    this.getAttributeValue = function(attribute, defaultValue, interpolate)
    {
        return angular.isDefined(attribute) ? (interpolate ? $interpolate(attribute)($scope.$parent) : $scope.$parent.$eval(attribute)) : defaultValue;
    };

    this.render = function()
    {
        this.page = parseInt($scope.page, 10) || 1;
        if (this.page > 0 && this.page <= $scope.totalPages)
        {
            $scope.pages = this.getPages(this.page, $scope.totalPages);
        }
    };

    $scope.selectPage = function(page)
    {
        if (!self.isActive(page) && page > 0 && page <= $scope.totalPages)
        {
            $scope.page = page;
            $scope.onSelectPage({ page: page });
        }
    };

    $scope.$watch('page', function()
    {
        self.render();
    });

    $scope.$watch('totalItems', function()
    {
        $scope.totalPages = self.calculateTotalPages();
    });

    $scope.$watch('totalPages', function(value)
    {
        setNumPages($scope.$parent, value); // Readonly variable

        if (self.page > value)
        {
            $scope.selectPage(value);
        } else
        {
            self.render();
        }
    });
} ])

.constant('paginationConfig', {
    itemsPerPage: 10,
    boundaryLinks: false,
    directionLinks: true,
    firstText: 'First',
    previousText: 'Previous',
    nextText: 'Next',
    lastText: 'Last',
    rotate: true
})

.directive('pagination', ['$parse', 'paginationConfig', '$templateCache', function($parse, config, $templateCache)
{
    return {
        restrict: 'EA',
        scope: {
            page: '=',
            totalItems: '=',
            onSelectPage: ' &'
        },
        controller: 'PaginationController',
        //templateUrl: 'template/pagination/pagination.html',
        template: $templateCache.get('template/pagination/pagination.html'),
        replace: true,
        link: function(scope, element, attrs, paginationCtrl)
        {

            // Setup configuration parameters
            var maxSize,
      boundaryLinks = paginationCtrl.getAttributeValue(attrs.boundaryLinks, config.boundaryLinks),
      directionLinks = paginationCtrl.getAttributeValue(attrs.directionLinks, config.directionLinks),
      firstText = paginationCtrl.getAttributeValue(attrs.firstText, config.firstText, true),
      previousText = paginationCtrl.getAttributeValue(attrs.previousText, config.previousText, true),
      nextText = paginationCtrl.getAttributeValue(attrs.nextText, config.nextText, true),
      lastText = paginationCtrl.getAttributeValue(attrs.lastText, config.lastText, true),
      rotate = paginationCtrl.getAttributeValue(attrs.rotate, config.rotate);

            paginationCtrl.init(config.itemsPerPage);

            if (attrs.maxSize)
            {
                scope.$parent.$watch($parse(attrs.maxSize), function(value)
                {
                    maxSize = parseInt(value, 10);
                    paginationCtrl.render();
                });
            }

            // Create page object used in template
            function makePage(number, text, isActive, isDisabled)
            {
                return {
                    number: number,
                    text: text,
                    active: isActive,
                    disabled: isDisabled
                };
            }

            paginationCtrl.getPages = function(currentPage, totalPages)
            {
                var pages = [];

                // Default page limits
                var startPage = 1, endPage = totalPages;
                var isMaxSized = (angular.isDefined(maxSize) && maxSize < totalPages);

                // recompute if maxSize
                if (isMaxSized)
                {
                    if (rotate)
                    {
                        // Current page is displayed in the middle of the visible ones
                        startPage = Math.max(currentPage - Math.floor(maxSize / 2), 1);
                        endPage = startPage + maxSize - 1;

                        // Adjust if limit is exceeded
                        if (endPage > totalPages)
                        {
                            endPage = totalPages;
                            startPage = endPage - maxSize + 1;
                        }
                    } else
                    {
                        // Visible pages are paginated with maxSize
                        startPage = ((Math.ceil(currentPage / maxSize) - 1) * maxSize) + 1;

                        // Adjust last page if limit is exceeded
                        endPage = Math.min(startPage + maxSize - 1, totalPages);
                    }
                }

                // Add page number links
                for (var number = startPage; number <= endPage; number++)
                {
                    var page = makePage(number, number, paginationCtrl.isActive(number), false);
                    pages.push(page);
                }

                // Add links to move between page sets
                if (isMaxSized && !rotate)
                {
                    if (startPage > 1)
                    {
                        var previousPageSet = makePage(startPage - 1, '...', false, false);
                        pages.unshift(previousPageSet);
                    }

                    if (endPage < totalPages)
                    {
                        var nextPageSet = makePage(endPage + 1, '...', false, false);
                        pages.push(nextPageSet);
                    }
                }

                // Add previous & next links
                if (directionLinks)
                {
                    var previousPage = makePage(currentPage - 1, previousText, false, paginationCtrl.noPrevious());
                    pages.unshift(previousPage);

                    var nextPage = makePage(currentPage + 1, nextText, false, paginationCtrl.noNext());
                    pages.push(nextPage);
                }

                // Add first & last links
                if (boundaryLinks)
                {
                    var firstPage = makePage(1, firstText, false, paginationCtrl.noPrevious());
                    pages.unshift(firstPage);

                    var lastPage = makePage(totalPages, lastText, false, paginationCtrl.noNext());
                    pages.push(lastPage);
                }

                return pages;
            };
        }
    };
} ])

.constant('pagerConfig', {
    itemsPerPage: 10,
    previousText: '« Previous',
    nextText: 'Next »',
    align: true
})

.directive('pager', ['pagerConfig', '$templateCache', function(config, $templateCache)
{
    return {
        restrict: 'EA',
        scope: {
            page: '=',
            totalItems: '=',
            onSelectPage: ' &'
        },
        controller: 'PaginationController',
        //templateUrl: 'template/pagination/pager.html',
        template: $templateCache.get('template/pagination/pager.html'),
        replace: true,
        link: function(scope, element, attrs, paginationCtrl)
        {

            // Setup configuration parameters
            var previousText = paginationCtrl.getAttributeValue(attrs.previousText, config.previousText, true),
      nextText = paginationCtrl.getAttributeValue(attrs.nextText, config.nextText, true),
      align = paginationCtrl.getAttributeValue(attrs.align, config.align);

            paginationCtrl.init(config.itemsPerPage);

            // Create page object used in template
            function makePage(number, text, isDisabled, isPrevious, isNext)
            {
                return {
                    number: number,
                    text: text,
                    disabled: isDisabled,
                    previous: (align && isPrevious),
                    next: (align && isNext)
                };
            }

            paginationCtrl.getPages = function(currentPage)
            {
                return [
          makePage(currentPage - 1, previousText, paginationCtrl.noPrevious(), true, false),
          makePage(currentPage + 1, nextText, paginationCtrl.noNext(), false, true)
        ];
            };
        }
    };
} ]);

angular.module("template/modal/backdrop.html", []).run(["$templateCache", function($templateCache)
{
    $templateCache.put("template/modal/backdrop.html",
    "<div class=\"modal-backdrop fade\" ng-class=\"{in: animate}\" ng-style=\"{'z-index': 1041 + index*10}\" ng-click=\"close($event)\"></div>");
} ]);

angular.module("template/modal/window.html", []).run(["$templateCache", function($templateCache)
{//fix by jutlin
    $templateCache.put("template/modal/window.html",
    "<div class=\"modal-scrollable\" ng-style=\"{'z-index': 1050 + index*10}\" ng-click=\"close($event)\"><div class=\"modal fade {{ windowClass }}\" ng-class=\"{in: animate}\" ng-style=\"{'z-index': 1051 + index*10}\" ng-transclude ng-click=\"$event.stopPropagation()\"></div></div>");
} ]);

angular.module("template/pagination/pager.html", []).run(["$templateCache", function($templateCache)
{
    $templateCache.put("template/pagination/pager.html",
    "<div class=\"pager\">\n" +
    "  <ul>\n" +
    "    <li ng-repeat=\"page in pages\" ng-class=\"{disabled: page.disabled, previous: page.previous, next: page.next}\"><a ng-click=\"selectPage(page.number)\">{{page.text}}</a></li>\n" +
    "  </ul>\n" +
    "</div>\n" +
    "");
} ]);

angular.module("template/pagination/pagination.html", []).run(["$templateCache", function($templateCache)
{
    $templateCache.put("template/pagination/pagination.html",
    "<div class=\"pagination\"><ul class=\"pagination\">\n" +
    "  <li ng-repeat=\"page in pages\" ng-class=\"{active: page.active, disabled: page.disabled}\"><a ng-click=\"selectPage(page.number)\">{{page.text}}</a></li>\n" +
    "  </ul>\n" +
    "</div>\n" +
    "");
} ]);
