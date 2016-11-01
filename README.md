# docto
Docto is a domain specific language for document-oriented database iteractions in Elixir.

## Document

defmodule Sample.Droid do
  use Docto.Document
  
  field :object,   :id # Database specific global document id
  field :city      # Defaults type will be casted automatically
  field :temp_hi,  :integer
  field :prcp,     :float, default: 0.0
  field :firmware, :binary # Database specific binary storage
  field :docs, :document # Another Docto.Document or [{"key", "value"}] | %{"key" => "value"}
  
  
  validate do
    required: [:name, :email],
    format: {:email, ~r/@/},
    inclusion: {:age, 18..20}
  end
  
end

defmodule Sample.App do
  import Docto.Query
  alias Sample.Droid
  alias Sample.Repo

  def keyword_query do
    Droid.
         where: w.prcp > 0 or is_nil(w.prcp),
         select: w
    Repo.all(query)
  end

  def pipe_query do
    Weather
    |> where(city: "Kraków")
    |> order_by(:temp_lo)
    |> limit(10)
    |> Repo.all
  end
end

## Data representation

    BSON                Elixir
    ----------        	------
    double              0.0
    string              "Elixir"
    document            [{"key", "value"}] | %{"key" => "value"} (1)
    binary              %BSON.Binary{binary: <<42, 43>>, subtype: :generic}
    object id           %BSON.ObjectId{value: <<...>>}
    boolean             true | false
    UTC datetime        %BSON.DateTime{utc: ...}
    null                nil
    regex               %BSON.Regex{pattern: "..."}
    JavaScript          %BSON.JavaScript{code: "..."}
    integer             42
    symbol              "foo" (2)
    min key             :BSON_min
    max key             :BSON_max

1) Since BSON documents are ordered Elixir maps cannot be used to fully represent them. This driver chose to accept both maps and lists of key-value pairs when encoding but will only decode documents to lists. This has the side-effect that it's impossible to discern empty arrays from empty documents. Additionally the driver will accept both atoms and strings for document keys but will only decode to strings.

2) BSON symbols can only be decoded.
