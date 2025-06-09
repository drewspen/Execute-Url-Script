# Execute-Url-Script
GTM tag template tpl used to Execute Url Script tag

For use when the GTM custom HTML equivalents match the following. 

-- Survicate
<script type='text/javascript'> (function(w) { var s = document.createElement('script'); s.src = {{Survicate Source URL}}; s.async = true; var e = document.getElementsByTagName('script')[0]; e.parentNode.insertBefore(s, e); })(window); 
</script>
