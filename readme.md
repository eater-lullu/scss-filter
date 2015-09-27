#Basic usage

	@include add-filter('unicorns', 'rainbow');
	@function rainbow($value) {
		@return $value + 'on rainbows';
	}

Always keep in mind that the add-filter mixin must be called before the apply-filter function!
	
	$unicorns: 'are awesome';
	$unicorns: apply-filter('unicorns', $unicorns);

	@debug $unicorns;
	// $unicorns: 'are awesome on rainbows';

##Register a new filter

Use the apply-filter function to register a new filter.

	/// Call functions registerd to the tag
	/// 
	/// @author Lukas Kleinschmidt
	/// 
	/// @require {Map} $scss-filter !global
	/// 
	/// @param  {String} $tag
	/// @param  {Mixed} $value
	/// @param  {ArgList} $var...
	/// 
	/// @return {Mixed}
	@function apply-filter($tag, $value, $var...) {

	  @if global-variable-exists('scss-filter') {

	    @if map-has-key($scss-filter, $tag) {

	      $priorities: map-get($scss-filter, $tag);

	      @while length($priorities) > 0 {

	        $keys: map-keys($priorities);
	        $key: min($keys...);

	        $functions: map-get($priorities, $key);
	        $priorities: map-remove($priorities, $key);

	        // Loop through alle functions
	        @each $function in $functions {

	          @if function-exists($function) {

	            @if length($var) == 0 {
	              $value: call($function, $value);
	            }

	            @else {
	              $value: call($function, $value, $var);
	            }
	          }
	        }
	      }
	    }
	  }

	  @return $value;
	}

##Add a filter function

Use the add-filter mixin to modify values passed by the apply-filter function.

	/// Mixin to add a scss-filter
	/// 
	/// @author Lukas Kleinschmidt
	/// 
	/// @require {Map} $scss-filter !global
	/// 
	/// @param  {String} $tag
	/// @param  {String} $function
	@mixin add-filter($tag, $function, $priority: 10) {

	  @if not(global-variable-exists('scss-filter')) {

	    /// Global variable to keep track
	    /// of all filters
	    /// 
	    /// @type {Map}
	    /// 
	    /// @example $scss-filter: (
	    ///   $tag: (
	    ///     $priority: (
	    ///       $function...
	    ///     )
	    ///   )
	    /// )
	    $scss-filter: () !global;
	  }

	  // tag is already registered
	  @if map-has-key($scss-filter, $tag) {

	    $priorities: map-get($scss-filter, $tag);
	    
	    // priority is already registered
	    @if map-has-key($priorities, $priority) {

	      // merge $function with already registered functions
	      $functions: map-get($priorities, $priority);
	      $functions: append($functions, $function);
	      $function: $functions;
	    }

	    // merge $priority with already registered priorities
	    $priorities: map-merge($priorities, ($priority: $function));
	    $priority: $priorities;
	  }

	  @else {

	    // map $function to $priority
	    $priority: ($priority: $function);
	  }

	  // merge new $scss-filter with $scss-filter
	  $scss-filter: map-merge($scss-filter, ($tag: $priority)) !global;
	}
